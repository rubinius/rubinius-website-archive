---
layout: doc_de
title: Umstellung von MRI zu Rubinius
previous: Anleitungen
previous_url: guides
next: How-To
next_url: how-to
---

Diese Anleitung soll dir bei der Umstellung deiner Ruby oder Ruby on Rails
Anwendung von MRI (Matz Ruby Interpreter) zu Rubinius helfen. Dabei wird davon
ausgegangen, dass dir Grundlegende Begriffe und Programme wie Ruby, Ruby on
Rails, Rubygems und Bundler bekannt sind. Ebenso wird angenommen, dass du dich
im Terminal zurechtfindest.

## 1. Rubinius installieren

Der erste Schritt zum erfolgreichen Umstieg auf Rubinius ist die Installation
von Rubinius. Da du beim Umstieg  notwendigerweises zwischen MRI und Rubinius
hin- und herwechseln willst, bietet es sich an, eines der Hilfsprogramme zu
installieren, das den schnellen Wechsel zwischen unterschiedlichen
Ruby-Versionen ermöglicht.

Zur Verwendung mit Rubinius wird an dieser Stelle das Kommandozeilenprogramm
[chruby](https://github.com/postmodern/chruby) empfohlen.

### chruby und ruby-install installieren

Verwendest du Mac OS X, ist es am einfachsten `chruby`und `ruby-install` mit
Hilfe des [Homebrew](https://github.com/Homebrew/homebrew) Paket Managers zu
installieren:

    $ brew update
    $ brew install chruby ruby-install

Möchtest Du Homebrew nicht verwenden oder benutzt ein anderes Betriebssystem,
dann schlage bitte weitere Installationsmöglichkeiten unter
[chruby Installationshandbuch](https://github.com/postmodern/chruby#install) und
dem [ruby-install Handbuch](https://github.com/postmodern/ruby-install#install)
nach.

Um `chruby` zu konfigurieren, füge die folgende Zeile zu deiner ~/.bashrc oder
~/.zshrc:

    source /usr/local/opt/chruby/share/chruby/chruby.sh

Beachte das Handbuch für weitere
[Einstellungsmöglichkeiten](https://github.com/postmodern/chruby#configuration).

### ruby-install verwenden

Sobald du `ruby-install` eingerichtet hast, kann Rubinius installiert werden:

    $ ruby-install rbx 2.2.5

Für Hilfen zur Installation weiterer Ruby-Implementierungen mit `ruby-install`,
sei an dieser Stelle auf die
[Installationshinweise](https://github.com/postmodern/ruby-install#synopsis)
verwiesen.

### chruby verwenden

Sobald Rubinius installiert ist, kann es mit folgenden Befehl aktiviert werden:

    $ chruby rbx

## 2. Gems

Die meisten Gems, die unter MRI laufen, sollten auch unter Rubinius laufen. Den
Ausnahmen ist ein besonderer Abschnitt weiter unten gewidmet. Darüber hinaus
stellt Rubinius viele Systemkomponenten als zusätzliche Gems zur Verfügung.
Darunter z.B. die Komponenten, die in Rubinius für das Parsen oder die
Bytecodekompilierung zuständig sind. Ebenso stehen ein Debugger, ein Profiler
und die Ruby Standardbibliothek zur Verfügung. Diese Gems werden mit Rubinius
vorinstalliert.

### C-Erweiterungs-Gems

Viele Gems, die C-Erweiterungen verwenden, laufen problemlos unter Rubinius.
Davon ausgenommen sind Gems, die auf interne Datenstrukturen von MRI
zurückgreifen. Diese Gems können von Rubinius nicht unterstützt werden. Davon
betroffen sind gems wie `ruby-debug` oder `ruby-prof`. Rubinius bringt dafür,
wie oben erwähnt, einen eigenen Debugger und Profiler mit. Darüber hinaus auch
noch weitere Hilfsprogramme. Eine Übersicht befindet sich auf den Seiten der
[Hilfsprogramme](http://rubini.us/doc/de/tools/).

## 3. Gemfiles

Dein Gemfile sollte problemlos unter Rubinius laufen. Zuvor solltest du aber ein
`bundle update` durchführen, damit alle Gem-Abhängigkeiten unter Rubinius erneut
aufgelöst werden können. Werden Gems verwendet, die mit Rubinius inkompatibel
sind, so können sie bis zur vollständigen Umstellung deiner Anwendung auf
Rubinius in einem `platforms`-Block aufgeführt werden:

    # Beispiel für einen platforms Block für MRI-spzifische Gems
    platforms :mri do
      gem 'ruby-prof'
      gem 'ruby-debug'
    end

## 4. Kompatibilitätsprobleme

Rubinius hat das [RubySpec](http://rubyspec.org) Projekt ins Leben gerufen und
erweitert es kontinuierlich. Es beschreibt das Verhalten von Ruby und beobachtet
die Kompatibilität von Rubinius (wie auch anderer Implemtationen) zu MRI.

Mit wenigen Ausnahmen ist Rubinius vollständig kompatibel zu MRI 2.1. Einige
Funktionen, wie z.B. die sogenannten `keyword argumets`, sind bislang noch nicht
implementiert. Andere Funktionen sind möglicherweise ebenfalls nicht
implementiert, wenn sie z.B. unbekannt sind oder keine RubySpecs dazu
existieren. Einige Komponenten der Standardbibliothek sind ebenfalls noch nicht
implementiert, davon sind z.B. Continuation Ripper, TracePoint und Tracer
betroffen. Möglicherweise wird dies in Zukunf noch nachgeholt.

Sollte dir ein inkompatibles Verhalten von Rubinius im Vergleich zu MRI
auffallen, ist es höchstwahrscheinlich ein Fehler und sollte [gemeldet
werden](https://github.com/rubinius/rubinius/issues).

