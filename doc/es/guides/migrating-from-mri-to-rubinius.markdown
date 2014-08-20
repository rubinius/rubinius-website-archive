---
layout: doc_es
title: Guías - Migrando de MRI a Rubinius
previous: Guías
previous_url: guides
next: Cómo
next_url: how-to
---

Esta guía te acompañará en el proceso de migrar tu aplicación Ruby on Rails
de MRI (Matz's Ruby Implementation) a Rubinius. Esta guía asume que estás
familiarizado con Ruby o Ruby on Rails, RubyGems, y Bundler. También asume que
sabes utilizar el shell de la línea de comandos.

## 1. Instalar Rubinius

El primer paso en la migración a Rubinius es instalar Rubinius. Ya que
probablemente necesitarás alternar entre MRI y Rubinius durante la migración,
la solución más práctica es utilizar una utilidad para cambiar de Ruby.

Te recomendamos usar [chruby](https://github.com/postmodern/chruby) como
herramienta para la línea de comandos que permite cambiar entre las
implementaciones de Ruby. Todos los ejemplos de esta guía usarán `chruby`.

### Instalar chruby y ruby-install

Si estás usando OS X, la forma más fácil de instalar `chruby` y
`ruby-install` es usar el administrador de paquetes [Homebrew](https://github.com/Homebrew/homebrew)
como se describe a continuación:

    $ brew update
    $ brew install chruby ruby-install

Si no usas Homebrew o usas un sistema operativo diferente a OS X, por favor
refiérete a la siguiente documentación:

1. [Instrucciones para instalar chruby](https://github.com/postmodern/chruby#install)
2. [Instrucciones para instalar ruby-install](https://github.com/postmodern/ruby-install#install)

### Habilitar chruby

Para habilitar `chruby`, agrega lo siguiente a tu archivo ~/.bashrc o ~/.zshrc:

    source /usr/local/opt/chruby/share/chruby/chruby.sh

Refiérete al a documentación para encontrar más [instrucciones de configuración](https://github.com/postmodern/chruby#configuration).

### Usar ruby-install

Una vez instales la utilidad `ruby-install`, instalar Rubinius es bastante
sencillo:

    $ ruby-install rbx 2.2.6

Para instrucciones sobre como instalar otras implementaciones de Ruby, por
favor refiérete al [resumen de instalación](https://github.com/postmodern/ruby-install#synopsis)
de `ruby-install`.

### Usar chruby

Ahora que Rubinius está instalado, puedes activarlo con el siguiente comando:

    $ chruby rbx

## 2. Gemas

La mayoría de las gemas que corren sobre MRI deberían correr sobre Rubinius,
con excepción de las descritas más adelante. Adicionalmente, Rubinius pone
muchos componentes del sistema disponibles como gemas. Estos incluyen
herramientas de análisis y compilación de código Ruby, asía como el 
*debugger* y el *profiler*. Todas esas gemas son pre-instaladas cuando instalas
Rubinius.

### Gemas con C-extensions (extensiones de C)

Muchas gemas que usan extensiones de C corren bien sobre Rubinius. La 
excepción son las gemas que dependen de las estructuras de datos internas de
MRI. Estas gemas no pueden ser soportadas en Rubinius, entre las que se 
incluyen gemas como `ruby-debug` y `ruby-prof`. Rubinius provee sus propias
herramientas, tales como como su propio *debugger* y *profiler*. Para conocer
más detalles refiérete a la [documentación de las herramientas](http://rubini.us/doc/en/tools/).

## 3. Gemfiles

Tu Gemfile deberían funcionar bien con Rubinius pero deberías ejecutar 
`bundle update` para forzar que se vuelvan a calcular las dependencias de las 
gemas para Rubinius. Adicionalmente si estás usando gemas que son
incompatibles con Rubinius, puedes ponerlas en un bloque llamado `platforms`
hasta que completes la migración a Rubinius como se muestra a continuación:

    # Ejemplo de un bloque con gemas específicas a la plataforma MRI
    platforms :mri do
      gem 'ruby-prof'
      gem 'ruby-debug'
    end

## 4. Problemas de Compatibilidad

Rubinius creó el proyecto [RubySpec](http://rubyspec.org), el cual mejora
continuamente, con el objetivo de describir el comportamiento de Ruby y 
monitorear la compatibilidad con MRI.

Fuera de unas pocas excepciones se espera que Rubinius sea compatible con la
versión 2.1 de MRI. Algunas características como los argumentos *keyword*, no
han sido implementados aún. Puede que otras características no hayan sido
implementadas debido que son desconocidas y no existes RubySpecs para ellas 
aún. Varios componentes de la librería estándar, incluyendo Continuation, 
Ripper, TracePoint, y Tracer, no han sido implementados aún pero pueden ser
implementados en el futuro.

Si encuentras algún comportamiento incompatible en Rubinius en comparación
con MRI, probablemente sea un *bug*. Por favor [crea un issue](https://github.com/rubinius/rubinius/issues)
para reportarlo.
