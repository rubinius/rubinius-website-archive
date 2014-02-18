---
layout: doc_it
title: Guide - Migrare da MRI a Rubinius
previous: Guide
previous_url: guides
next: How-To
next_url: how-to
---

Questa guida vi aiuterà nella migrazione della vostra applicazione Ruby o Ruby
on Rails da MRI (Matz's Ruby Implementation) a Rubinius. La guida assume che
abbiate familiarità con Ruby o Ruby on Rails, RubyGems e Bundler. Si suppone
inoltre che sappiate usare la riga di comando.

## 1. Installare Rubinius

Il primo passo della migrazione a Rubinius è installare Rubinius. Durante la
migrazione avrete probabilmente bisogno di passare spesso da MRI a Rubinius e
viceversa, vi suggeriamo quindi di utilizzare uno strumento che vi consenta di
farlo.

Lo strumento che raccomandiamo è
[chruby](https://github.com/postmodern/chruby), che utilizzeremo per tutti gli
esempi di questa guida.

### Installare chruby e ruby-install

Se usate OS X, il modo più facile per installare `chruby` e `ruby-install` è
usare [Homebrew](https://github.com/Homebrew/homebrew):

    $ brew update
    $ brew install chruby ruby-install

Se non utilizzate Homebrew o utilizzate un sistema operativo diverso da OS X,
fate riferimento alle [istruzioni di installazione di
chruby](https://github.com/postmodern/chruby#install) e le [istruzioni di
installazione di
ruby-install](https://github.com/postmodern/ruby-install#install).

Per configurare `chruby`, aggiungete questa riga al file ~/.bashrc o ~/.zshrc:

    source /usr/local/opt/chruby/share/chruby/chruby.sh

Consultate le [istruzioni di
configurazione](https://github.com/postmodern/chruby#configuration) per
maggiori informazioni.

### Utilizzare ruby-install

Una volta installato `ruby-install`, installare Rubinius è semplicissimo:

    $ ruby-install rbx 2.2.5

Per istruzioni su come installare altre implementazioni di Ruby, fate
riferimento alla
[documentazione](https://github.com/postmodern/ruby-install#synopsis).

### Utilizzare chruby

Ora che Rubinius è installato, potete attivarlo con questo comando:

    $ chruby rbx

## 2. Gemme

La maggior parte delle gemme che funzionano su MRI dovrebbero funzionare anche
su Rubinius, con le eccezioni evidenziate di seguito. Inoltre Rubinius rende
dei componenti del sistema disponibili come gemme: queste comprendono gli
strumenti usati da Rubinius per effettuare il parsing e la compilazione di
codice Ruby, il debugger, il profiler e la Ruby standard library. Tutte queste
gemme vengono pre-installate quando si installa Rubinius.

### Gemme con estensioni in C

Molte gemme che utilizzano estensioni in C funzionano perfettamente su
Rubinius. Fanno eccezione quelle che dipendono da strutture dati interne ad
MRI: queste gemme non possono essere supportate su Rubinius, e includono
`ruby-debug` e `ruby-prof`. Rubinius fornisce però un suo debugger e un suo
profiler, oltre ad altri strumenti. Maggiori dettagli nella
[documentazione](http://rubini.us/doc/en/tools/).

## 3. Gemfile

Il vostro Gemfile dovrebbe funzionare con Rubinius, ma è necessario eseguire
`bundle update` per forzare il ricalcolo delle dipendenze. Se utilizzate gemme
che non sono compatibili con Rubinius, potete inserirle in un blocco
`platforms` finché la migrazione a Rubinius non sarà completata:

    # Esempio di blocco `platforms`
    platforms :mri do
      gem 'ruby-prof'
      gem 'ruby-debug'
    end

## 4. Problemi di compatibilità

La comunità di Rubinius ha dato vita al progetto
[RubySpec](http://rubyspec.org), il cui obiettivo è descrivere il comporamento
di Ruby e monitorare la compatibilità delle diverse implementazioni di Ruby con
MRI.

Con qualche eccezione, Rubinius dovrebbe essere compatibile con MRI 2.1. Alcune
funzionalità, come i keyword arguments, non sono ancora state implementate.
Altre funzionalità potrebbero non essere state implementate in quanto
sconosciute e non ancora coperte da RubySpec. Numerosi componenti della
standard library, come Continuation, Ripper, TracePoint, e Tracer, non sono
ancora presenti ma potrebbero esserlo in futuro.

Se riscontrate un'incompatibilità tra il comportamento di Rubinius e quello di
MRI, probabilmente si tratta di un bug. Per favore
[segnalatelo](https://github.com/rubinius/rubinius/issues).
