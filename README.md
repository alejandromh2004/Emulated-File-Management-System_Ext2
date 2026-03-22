# Sistema de Ficheros Ext2 Simulado

Este proyecto consiste en la implementación de un sistema de ficheros similar a **ext2** (Second Extended File System) desde cero, desarrollado en lenguaje C. El objetivo es comprender en profundidad el funcionamiento interno de un sistema de archivos: estructuras de datos (superbloque, inodos, bloques de datos), gestión de directorios, creación y borrado de enlaces físicos (*hard links*), manipulación de permisos, y operaciones de lectura/escritura.

La práctica se ha dividido en varios niveles de complejidad creciente, culminando en una interfaz de comandos que permite interactuar con el sistema de ficheros como si se tratara de un disco real.

---

## Tabla de Contenidos
- [Características principales](#características-principales)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Compilación](#compilación)
- [Uso de los comandos](#uso-de-los-comandos)
  - [Gestión del disco](#gestión-del-disco)
  - [Creación de directorios y ficheros](#creación-de-directorios-y-ficheros)
  - [Listado de contenidos](#listado-de-contenidos)
  - [Cambio de permisos](#cambio-de-permisos)
  - [Información de inodos](#información-de-inodos)
  - [Escritura y lectura de ficheros](#escritura-y-lectura-de-ficheros)
  - [Enlaces físicos](#enlaces-físicos)
  - [Borrado de elementos](#borrado-de-elementos)
- [Estructuras internas clave](#estructuras-internas-clave)
- [Detalles de implementación](#detalles-de-implementación)
- [Pruebas](#pruebas)
- [Autores](#autores)

---

## Características principales

* Sistema de ficheros tipo ext2 con bloques de 1024 bytes.
* Gestión de metadatos mediante inodos (tipo, permisos, tamaño, marcas de tiempo, contador de enlaces).
* Directorios como ficheros especiales que contienen entradas (nombre + número de inodo).
* Enlaces físicos (*hard links*) que permiten múltiples nombres para un mismo inodo.
* Caché de directorios para optimizar el acceso a rutas (última entrada leída/escrita).
* Comandos en línea similares a los de Unix: `mkdir`, `touch`, `ls`, `chmod`, `stat`, `cat`, `ln`, `rm`, etc.
* Soporte para permisos de lectura/escritura (r/w) en inodos.
* Recuperación de espacio al eliminar ficheros o enlaces (liberación de inodos y bloques).
* Manejo de errores con códigos específicos y mensajes informativos.

---

## Estructura del proyecto

El proyecto está organizado en varias capas y ficheros:

```text
.
├── bloques.c / .h           # Gestión de bloques del dispositivo (mount, read, write)
├── ficheros_basico.c / .h   # Operaciones básicas sobre inodos (leer, escribir, reservar, liberar)
├── ficheros.c / .h          # Operaciones sobre ficheros (mi_write_f, mi_read_f, mi_truncar_f)
├── directorios.c / .h       # Capa de directorios: búsqueda de entradas, enlaces, borrado
├── mi_mkfs.c                # Formateo del disco (creación del superbloque y array de inodos)
├── leer_sf.c                # Lectura del superbloque (debug)
├── mi_mkdir.c               # Creación de directorios/ficheros (mi_creat)
├── mi_touch.c               # (Opcional) Creación de ficheros
├── mi_ls.c                  # Listado de directorios/ficheros (mi_dir)
├── mi_chmod.c               # Cambio de permisos (mi_chmod)
├── mi_stat.c                # Mostrar metadatos de un inodo (mi_stat)
├── mi_escribir.c            # Escritura en un fichero (mi_write)
├── mi_cat.c                 # Lectura de un fichero (mi_read)
├── mi_link.c                # Creación de enlaces físicos (mi_link)
├── mi_rm.c                  # Borrado de ficheros/directorios (mi_unlink)
├── mi_rmdir.c               # (Opcional) Borrado específico de directorios
├── prueba_cache_tabla.c     # Prueba de la caché de directorios (FIFO/LRU)
├── Makefile                 # Compilación automatizada
└── test[7-10].sh            # Scripts de prueba para cada nivel

Compilación

Para compilar todos los programas, simplemente ejecuta:
Bash

make

Si deseas compilar un programa específico, por ejemplo mi_mkfs:
Bash

gcc -o mi_mkfs mi_mkfs.c bloques.c ficheros_basico.c -Wall

    Nota: Se recomienda utilizar el Makefile incluido para evitar errores de dependencias.

Uso de los comandos

Todos los comandos reciben como primer parámetro el nombre del dispositivo virtual (fichero que simula el disco). El dispositivo debe ser formateado previamente con mi_mkfs.
Gestión del disco

Crear un disco virtual:
Bash

./mi_mkfs <nombre_dispositivo> <nbloques>

Ejemplo: ./mi_mkfs disco 100000 crea un disco de 100.000 bloques (unos 100 MB).
Creación de directorios y ficheros

Crear directorio:
Bash

./mi_mkdir <disco> <permisos> </ruta/directorio/>

(La ruta debe terminar en /. Los permisos son en octal, 0-7).

Crear fichero (si se usa mi_touch):
Bash

./mi_touch <disco> <permisos> </ruta/fichero>

(La ruta NO termina en /).
Listado de contenidos

Listar directorio o fichero:
Bash

./mi_ls <disco> </ruta>
./mi_ls -l <disco> </ruta>   # Formato extendido

Para un directorio, muestra las entradas; para un fichero, muestra sus metadatos en una línea.
Cambio de permisos

Modificar permisos:
Bash

./mi_chmod <disco> <permisos> </ruta>

Información de inodos

Mostrar metadatos del inodo:
Bash

./mi_stat <disco> </ruta>

Ejemplo de salida:
Plaintext

Nº de inodo: 3
tipo: f
permisos: 6
atime: Wed 2025-02-12 15:15:08
mtime: Wed 2025-02-12 15:15:08
ctime: Wed 2025-02-12 15:15:08
nlinks: 2
tamEnBytesLog: 11
numBloquesOcupados: 1

Escritura y lectura de ficheros

Escribir texto en un fichero:
Bash

./mi_escribir <disco> </ruta/fichero> "<texto>" <offset>

(Devuelve el número de bytes escritos).

Leer todo el contenido de un fichero (como cat):
Bash

./mi_cat <disco> </ruta/fichero> [> fichero_salida.txt]

(Si se añade > fichero.txt, la salida se redirige a un fichero externo).
Enlaces físicos

Crear un enlace físico (hard link):
Bash

./mi_link <disco> </ruta/origen> </ruta/destino>

(Ambos deben ser ficheros, no directorios. El destino no debe existir previamente).
Borrado de elementos

Eliminar un fichero o directorio vacío:
Bash

./mi_rm <disco> </ruta>

(Si es un directorio, debe estar vacío. El directorio raíz no se puede borrar).
Estructuras internas clave

Superbloque (struct superbloque): Almacena información global del sistema de ficheros (número total de bloques, inodos, posición del primer inodo libre, etc.).

Inodo (struct inodo): Contiene los metadatos de un fichero o directorio:
C

struct inodo {
    unsigned char tipo;                 // 'd' directorio, 'f' fichero
    unsigned char permisos;             // bits: 4 lectura, 2 escritura, 1 ejecución
    time_t atime, mtime, ctime, btime;  // tiempos
    unsigned int nlinks;                // contador de enlaces
    unsigned int tamEnBytesLog;         // tamaño lógico en bytes
    unsigned int numBloquesOcupados;
    unsigned int punterosDirectos[12];  // punteros a bloques de datos
    unsigned int punterosIndirectos[3]; // indirecto simple, doble, triple
};

Entrada de directorio (struct entrada):
C

struct entrada {
    char nombre[60];
    unsigned int ninodo;
};

(Cada bloque de directorio contiene 16 entradas: 1024 / 64).
Detalles de implementación

    Gestión de bloques: La capa bloques.c proporciona funciones bread y bwrite para leer/escribir bloques del dispositivo, así como bmount y bumount.

    Nivel básico de ficheros: ficheros_basico.c maneja la reserva y liberación de inodos y bloques, la lectura/escritura de inodos y la traducción de bloques lógicos a físicos.

    Nivel de ficheros: ficheros.c implementa mi_write_f y mi_read_f para escribir/leer datos de un inodo gestionando los bloques y los punteros directos/indirectos.

    Nivel de directorios: directorios.c contiene:

        extraer_camino(): descompone una ruta en inicial y final.

        buscar_entrada(): función recursiva que localiza una entrada (o la crea) comprobando permisos y existencia.

        mi_creat(): crea un nuevo fichero o directorio.

        mi_dir(): lista el contenido de un directorio (usado por mi_ls).

        mi_chmod(), mi_stat(): delegados a las funciones de ficheros.

        mi_write(), mi_read(): versiones que usan caché de directorios y llaman a mi_write_f/mi_read_f.

        mi_link(), mi_unlink(): para enlaces físicos y borrado.

    Caché de directorios: Se implementa una caché simple (última entrada) o una tabla FIFO/LRU para reducir las llamadas a buscar_entrada() en operaciones consecutivas sobre el mismo fichero.

    Manejo de errores: Los códigos de error se definen en directorios.h y se muestran con mostrar_error_buscar_entrada().

Pruebas

Se proporcionan scripts de prueba para cada nivel (test7.sh, test8.sh, test9.sh, test10.sh). Para ejecutar todas las pruebas:
Bash

./test10.sh

Estos scripts crean un disco, realizan operaciones y comprueban la salida esperada. También se incluye un programa prueba_cache_tabla.c para verificar el correcto funcionamiento de la caché de directorios con diferentes algoritmos de reemplazo.
Autores

    Adelaida Delgado (diseño original de la práctica).

    Implementación y adaptación realizada por Alejandro Martínez.

    Nota: Este proyecto es puramente académico y tiene como objetivo comprender los fundamentos de los sistemas de ficheros. No está diseñado para ser utilizado en entornos de producción.
