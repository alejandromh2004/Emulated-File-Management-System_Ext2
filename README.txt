Practica de S02
Grupo: Un momento
Integrantes: Javier Vivo Samaniego
             Alejandro Martínez Hermosa
             Martín Serra Rubio
Entrega final de la practica de la asignatura Sistemas Operativos II de la UIB (Universidad de las Islas Baleares), en esta
práctica se implementa un sistema de ficheros virtual que simula el ext2 capaz de hacer todas las funciones habituales de un 
sistema de ficheros (crear/borrar ficheros/directorios, leer y escribir ficheros con offsets a elegir, modificar permisos de 
entradas del sistema, etc.).
Como mejoras a destacar de la practica cabe recalcar:
    - Uso de una caché de direcciones de entradas recientemente usadas, tenemos una caché básica que guarda la ultima entrada
    tanto leida como escrita por el principio de localidad (prevee que se usará dentro de poco otra vez esa entrada).
    - Diferenciacion de crear directorios y crear ficheros, se podría haber dejado como un simple mkdir y cuando no tiene / final
    era un touch, pero se ha implementado el programa por separado.
    - El comando ls permite una version extendida "-l" con la que se pueden ver datos más profundos de la entrada referida.
Todas estas mejoras han sido implementadas en conjunto por el equipo entero.
