---
layout: doc_en
title: How-To - Obtaining GDB Backtraces
previous: Commit to Github
previous_url: how-to/commit-to-github
---

When reporting issues it's often requested that you provide the backtraces of a
running process from GDB. While Rubinius displays a Ruby backtrace upon a crash
it doesn't show a backtrace of the underlying C++ code, for this you'll need to
use GDB.

Assuming GDB is installed, run your application as following:

    gdb --args ruby path/to/application

In case your application is started using a Ruby executable not present in your
local directory (e.g. a Rails application) you can use the following instead:

    gdb --args ruby `which path/to/executable`

For example, for Rails:

    gdb --args ruby `which rails` console

Once GDB has started you'll be greeted with a prompt that looks something like
the following:

    GNU gdb (GDB) 7.8.2
    Copyright (C) 2014 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-unknown-linux-gnu".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
    <https://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
    <https://www.gnu.org/software/gdb/documentation/>.
    For help, type "help".
    Type "apropos word" to search for commands related to "word"...
    Reading symbols from ruby...done.
    (gdb)

The easiest way of obtaining (and sharing) GDB backtraces is to log them to a
file and upload them to <https://gist.github.com>. To do so run the following in
the GDB prompt:

    set logging on
    run

This will enable logging of GDB output to `./gdb.txt` and then run the actual
program. Upon a crash run the following in GDB:

    thread apply all bt

This will display the backtrace of all running threads in a pager. For example:

    Thread 8 (Thread 0x7f729513d700 (LWP 23389)):
    #0  0x00007f72942148cf in pthread_cond_wait@@GLIBC_2.3.2 () from /usr/lib/libpthread.so.0
    #1  0x0000000000893afd in wait (this=<optimized out>, mutex=...) at /home/yorickpeterse/Private/Projects/ruby/rubinius/vm/util/thread.hpp:448
    #2  worker_wait (this=<optimized out>) at vm/gc/finalize.cpp:422
    #3  rubinius::FinalizerThread::run (this=0x33e1c30, state=0x7f729513cec0) at vm/gc/finalize.cpp:144
    #4  0x000000000060b201 in rubinius::InternalThread::run (ptr=0x33e1c30) at vm/internal_threads.cpp:46
    #5  0x00007f729420f314 in start_thread () from /usr/lib/libpthread.so.0
    #6  0x00007f72933a424d in clone () from /usr/lib/libc.so.6

    Thread 7 (Thread 0x7f7291ea1700 (LWP 23390)):
    #0  0x00007f72942148cf in pthread_cond_wait@@GLIBC_2.3.2 () from /usr/lib/libpthread.so.0
    #1  0x00000000007d6e66 in wait (this=<optimized out>, mutex=...) at /home/yorickpeterse/Private/Projects/ruby/rubinius/vm/util/thread.hpp:448
    #2  rubinius::SignalThread::run (this=0x34118a0, state=0x7f7291ea0ec0) at vm/signal.cpp:109
    #3  0x000000000060b201 in rubinius::InternalThread::run (ptr=0x34118a0) at vm/internal_threads.cpp:46
    #4  0x00007f729420f314 in start_thread () from /usr/lib/libpthread.so.0
    #5  0x00007f72933a424d in clone () from /usr/lib/libc.so.6

    ...

To properly log the data to a file you must press enter a few times until you
reach the end of the output. Once done you can press `q` to quit GDB (this will
also terminate the process). Now you can upload the contents of `./gdb.txt` to
Gist.

For more information on GDB see
<https://www.gnu.org/software/gdb/documentation/>. For an example issue using GDB
logs see
<https://github.com/rubinius/rubinius/issues/2939#issuecomment-74933548>.
