---
layout: post
title: "A Small Case Study in Threading"
author: Chuck Remes
twitter: chuckremes
---

A recent project required me to process hundreds of CSV (comma-delimited) files and import them into a database. It's boring work, but it offer a few opportunities for me to utilize some of my favorite Ruby techniques and idioms.

The first version of the script was very simple.

1. Copy the zip file containg the CSVs to a working directory
2. Unzip the file
3. Delete unwanted files (e.g. README)
4. Loop through each filename
5. Convert the CSV lines where necessary (i.e. numbers should be Integer or Float instead of defaulting to String, do the same for dates)
6. Insert the converted data into the database
7. Delete the files as each completes, then go back to 1 for the next zip

I originally ran this under MRI and was getting approximately 3000 inserts per second on average. I did a quick back-of-the-envelope calculation and determined that the entire import would require about 15 days. Uh oh.

Ultimately I decided to write two scripts. The first script would handle Steps 1-4 and 7. The second script would handle Steps 5-6. Script One would launch Script Two as a subprocess via IO.popen and send filenames to Script Two's STDIN.

The real fun was in Script Two. Upon receiving a filename, it divides the file into multiple subfiles of equal size and generates an IO handle for each temp file. These IO handles are then passed to their own threads which do the processing described in Step 5. These threads batch up 10_000 lines and pass this bulk array to a dedicated database insertion thread which handles the bulk inserts. All inter-thread communication occurs through a SizedQueue. I chose a SizedQueue to provide backpressure to the parsing threads if they get too far ahead of the database thread.

Before spending any more time to optimize this work, I decided to benchmark the CSV parsing of a small file under MRI and Rubinius (JRuby is a whole different story worthy of its own post at the JRuby blog). The results are below.

```
GuestOSX:options_database cremes$ ruby -v
ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-darwin14]
GuestOSX:options_database cremes$ ruby benchmarks.rb 
Rehearsal ----------------------------------------------------------
parse CSV                7.910000   0.010000   7.920000 (  7.924954)
------------------------------------------------- total: 7.920000sec

                             user     system      total        real
parse CSV                8.040000   0.020000   8.060000 (  8.053098)

GuestOSX:options_database cremes$ chruby rbx
GuestOSX:options_database cremes$ ruby -v
rubinius 2.5.8 (2.1.0 bef51ae3 2015-09-24 3.5.1 JI) [x86_64-darwin14.5.0]
GuestOSX:options_database cremes$ ruby benchmarks.rb 
Rehearsal ----------------------------------------------------------
parse CSV               16.264571   0.161624  16.426195 ( 10.562584)
------------------------------------------------ total: 16.426195sec

                             user     system      total        real
parse CSV                9.084859   0.033108   9.117967 (  9.010402)
```
The test was single threaded. Looking at the "real" column the table shows that MRI is fastest with 8 seconds to parse 50_000 lines of my test data. Rubinius came in second place with 9 seconds to parse the same.

Running the test again with 4 threads each (on an 8-core machine) was enlightening.

```
GuestOSX:options_database cremes$ ruby -v
ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-darwin14]
GuestOSX:options_database cremes$ ruby benchmarks_multi.rb 
Rehearsal -------------------------------------------------------------
parse CSV - multithreaded   7.870000   0.100000   7.970000 (  7.962155)
---------------------------------------------------- total: 7.970000sec

                                user     system      total        real
parse CSV - multithreaded   7.930000   0.090000   8.020000 (  8.012822)

GuestOSX:options_database cremes$ chruby rbx
GuestOSX:options_database cremes$ ruby -v
rubinius 2.5.8 (2.1.0 bef51ae3 2015-09-24 3.5.1 JI) [x86_64-darwin14.5.0]
GuestOSX:options_database cremes$ ruby benchmarks_multi.rb 
Rehearsal -------------------------------------------------------------
parse CSV - multithreaded  20.991651   0.720063  21.711714 (  4.663789)
--------------------------------------------------- total: 21.711714sec

                                user     system      total        real
parse CSV - multithreaded  14.130549   0.136832  14.267381 (  2.984456)
```

MRI ran the 4-thread benchmark in the same 8 seconds as before! We are often reminded that MRI now maps its threads to native threads, but there is still a global interpreter lock (GIL) that prevents MRI from truly running code in parallel. Rubinius eliminated its GIL years ago, so all threads can run in parallel and produce a finishing time of just over 3 seconds.

With these improvements, Rubinius can finish my production job in about 5.5 days (versus the original 15 days). The CSV parsing work runs faster than the database can accept bulk inserts so, unless I want to spend a bunch of time optimizing the database configuration, my work is done. Thanks for Rubinius, I am saving 9 days on my import.

To reproduce these numbers on your own system, [the benchmarks and test data can be found here.](https://www.dropbox.com/s/6nv7j9n2r9ro771/csv-benchmarks.tgz?dl=0)

