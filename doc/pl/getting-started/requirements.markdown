---
layout: doc_pl
title: Wymagania
previous: Pierwsze kroki
previous_url: getting-started
next: Kompilacja
next_url: getting-started/building
---

Upewnij się, że masz zainstalowane poniższe programy i biblioteki
przed instalacją Rubiniusa. Poza tym przejrzyj podsekcje poniżej,
które opisują dokładne wymagania dla danego systemu operacyjnego.

Poniżej zostały przedstawione sugestie co do programów oraz bibliotek
wymaganych do skompilowania Rubiniusa. Twój system operacyjny lub
system zarządzania pakietami może udostępniać inne odpowiedniki tych bibliotek.

  * [GCC oraz G++ 4.x](https://gcc.gnu.org/)
  * [GNU Bison](https://www.gnu.org/software/bison/)
  * [MRI Ruby 2.0.0+](https://www.ruby-lang.org/) Jeśli nie masz
    zainstalowanego Ruby 2.0.0 w systemie rozważ skorzystanie z [RVM](https://rvm.beginrescueend.com/)
    aby go zainstalować.
  * [Rubygems](https://rubygems.org/)
  * [Git](https://git-scm.com/)
  * [ZLib](http://www.zlib.net/)
  * pthread - Biblioteka pthread powinna być zainstalowana przez Twój
    system operacyjny
  * [gmake](https://savannah.gnu.org/projects/make/)
  * [bundler](http://bundler.io/) `[sudo] gem install bundler`


### Apple OS X

Najprostszym sposobem przygotowania środowiska w systemie Apple OS X
jest zainstalowanie narzędzi XCode. Po zainstalowaniu, uaktywnij
"developer mode crash reporting": /Developer/Applications/Utilities/CrashReporterPrefs.app


### Debian/Ubuntu

  * ruby-dev lub ruby1.8-dev
  * libreadline5-dev
  * zlib1g-dev
  * libssl-dev
