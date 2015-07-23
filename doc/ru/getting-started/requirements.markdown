---
layout: doc_ru
title: Минимальные требования
previous: Для начала...
previous_url: getting-started
next: Сборка
next_url: getting-started/building
---

Убедитесь, что у вас установлены перечисленные программы и библиотеки. Также
просмотрите нижеприведенные разделы на предмет  требований для каждой
конкретной операционной системы.

Ниже следуют ссылки для получения дополнительной информации о программах и библиотеках,
нужных для сборки Rubinius. Для вашей операционной системы или менеджера
пакетов могут быть доступны и другие сборки, помимо перечисленных.

  * [GCC и G++ 4.x](https://gcc.gnu.org/)
  * [GNU Bison](https://www.gnu.org/software/bison/)
  * [MRI Ruby 2.0.0+](https://www.ruby-lang.org/) Если на вашей системе не
    установлен Ruby 2.0.0, подумайте об использовании
    [RVM](https://rvm.beginrescueend.com/) для его установки.
  * [Rubygems](https://rubygems.org/)
  * [Git](https://git-scm.com/)
  * [ZLib](http://www.zlib.net/)
  * pthread: Библиотека pthread должна быть установлена вашей операционной
    системой.
  * [gmake](https://savannah.gnu.org/projects/make/)
  * [rake](http://rake.rubyforge.org/): `[sudo] gem install rake`


### Mac OS X

Простейший путь для создания сборочного окружения на Mac OS X ---
установка XCode Tools and Utilities. После установки для формирования
отчетов об аварийных завершениях можно использовать приложение
/Developer/Applications/Utilities/CrashReporterPrefs.app


### Debian/Ubuntu

  * ruby-dev или ruby1.8-dev
  * libreadline5-dev
  * zlib1g-dev
  * libssl-dev

### FreeBSD

Под FreeBSD есть порт Rubinius, который называется `lang/rubinius`. Подробную
информацию о порте можно найти на сайте <https://freshports.org>. В процессе
инсталляции порта установка и настройка всех требуемых вспомогательных
программ и библиотек происходит автоматически.
