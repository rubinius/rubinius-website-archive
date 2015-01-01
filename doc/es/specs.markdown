---
layout: doc_es
title: Especificaciones
previous: Ruby - Variables Globales
previous_url: ruby/global-variables
next: Compiler
next_url: specs/compiler
---

El proyecto Rubinius utiliza especificaciones ejecutables al estilo
TDD/BDD para impulsar el desarrollo. El directorio 'spec' se divide
conceptualmente en dos partes:

   1. Todos los archivos en "./spec/ruby' que describen el comportamiento de
      MatzRuby.
   2. Y todos los otros archivos dentro del directorio './spec' que
      describen el comportamiento de Rubinius.

Las especificaciones que terminan en error se
etiquetan para que el proceso de intrgración continua (CI) utilize
siempre un conjunto de especificaciones que se sabe son exitosas. Esto
permite comprobar rapidamente que los cambios hechos al código de
Rubinius no causan regresiones.

Utilice el siguiente flujo para agregar especificaciones y código para Rubinius:

   1. Escriba una especificación no exitosa para algun aspecto del
      comportamiento de Ruby. Incluya los archivos correspondientes
      dentro de ./spec/ruby creando un commit para ellos.
   2. Agregue código a Rubinius para hacer pasar las especificaciones. Una vez
      más, haga commit de estos cambios por separado de las archivos
      de especificaciones.
   3. Ejecute el comando `rake` para asegurarse que todas las especificaciones
      anteriores y de integración continua siguen funcionando.

