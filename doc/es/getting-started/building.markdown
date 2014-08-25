---
layout: doc_es
title: Construyendo Rubinius
previous: Requisitos
previous_url: getting-started/requirements
next: Ejecutando Rubinius
next_url: getting-started/running-rubinius
---

Usted puede construir y ejecutar Rubinius desde el directorio
fuente. No es necesario instalar Rubinius en su sistema para poder
ejecutarlo.
Las instrucciones a continuación detallarán tanto la instalación de
Rubinius, como su ejecución desde el directorio fuente.

Rubinius utiliza LLVM para el compilador JIT y depende de LLVM 3.x. El `script`
de configuración revisará si alguna versión instalada, en caso de no ser 
así lo descargará. Si ha instalado LLVM en su sistema y por alguna razón
Rubinius no hace link con el, utilize la bandera `--skip-system` al ejecutar el
`script` de configuración que encontrará a continuación.

### Obtención del Código Fuente

El código fuente de Rubinius esta disponible como un archivo comprimido y como un
proyecto en Github.  Puede [descargar el archivo comprimido
aquí](https://github.com/rubinius/rubinius/tarball/master).

Para usar Git:


  1. Utilice la linea de comandos para entrar a su directorio de
  desarrollo.
  2. `git clone git://github.com/rubinius/rubinius.git`


### Instalación de Rubinius

Si planea utilizar Rubinius para ejecutar sus aplicaciones,
ésta es una buena opción. Sin embargo, también puede ejecutar Rubinius
directamente desde el directorio fuente. Vea la siguiente sección para
obtener más detalles al respecto.

Le recomendamos que instale Rubinius en un lugar que no requiera `sudo` o
privilegios de superusuario. Para instalar Rubinius:


  1. `./configure --prefix=/path/to/install/dir`
  2. `rake install`
  3. Siga las instrucciones para agregar el directorio _bin_ de Rubinius a su
     PATH

### Ejecución desde el directorio fuente

Si planea participar en el desarrollo de Rubinius, debe utilizar esta
opción.


  1. `./configure`
  2. `rake`

Si simplemente quire probar Rubinius, siga las instrucciones para
agregar el directorio _bin_ a su PATH.

Sin embargo, si está desarrollando Rubinius, usted NO debe agregar el
directorio _bin_ a su PATH porque el sistema de construcción de
Rubinius utilizara los vínculos de `ruby` y `rake` que utilizan el
ejecutable de Rubinius. Rubinius necesita
un ejecutable de Ruby aparte para poder reconstruirse a si mismo.

