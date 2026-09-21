Laboratorio Unidad 3 — Manejo del DEBUG
Este repositorio contiene el desarrollo de las dos partes del laboratorio guiado de manejo del DEBUG en DOSBox. La Parte 1 explora el estado inicial del procesador, la inspección de memoria con los comandos R, F, D y U, y la modificación puntual con E verificada mediante direccionamiento directo a memoria. La Parte 2 ensambla programas con el comando A, los ejecuta instrucción a instrucción con T y analiza el mecanismo de bucle LOOP frente a un bucle equivalente construido con DEC y JNZ.

Entorno utilizado

DOSBox 0.74-3 sobre Windows. El segmento asignado durante las sesiones fue 0724, por lo que las direcciones mostradas en las capturas aparecen como 0724:XXXX en lugar del 1357:XXXX del enunciado. El valor del segmento lo asigna el sistema en cada sesión.

Los directorios de trabajo se nombraron LAB3POS1 y LAB3POS2 porque el sistema de archivos del DOS limita los nombres a ocho caracteres y LAB3POST1 y LAB3POST2 se truncaban ambos al mismo nombre LAB3POST.

PARTE 1 — Exploración con DEBUG
Checkpoint 1 — Estado de registros
Captura: capturas/CP1_registros.png

Al iniciar DEBUG sin archivo, los cuatro registros de propósito general aparecen en cero, SP apunta a FFFE y los cuatro registros de segmento apuntan al mismo párrafo de memoria. El IP inicia en 0100, que es la primera dirección ejecutable después del PSP.

Al modificar AX con R AX y el valor 1234, la siguiente invocación de R muestra únicamente ese registro con el valor nuevo y el resto sin cambio, lo que confirma que la modificación es selectiva.

Checkpoint 2 — Volcado hexadecimal
Captura: capturas/CP2_volcado_memoria.png
Al observar la salida del comando D se identifican tres columnas. La primera columna corresponde a la dirección donde está ubicada esa línea en la memoria, esta se escribe en dos partes separada por dos puntos. El 0724 corresponde al segmento y el 0200 es el desplazamiento. La segunda columna es el contenido real de la memoria, compuesto por 16 bytes, donde cada uno son dos dígitos hexadecimales. La tercera columna es la interpretación ASCII, los valores que se encuentran fuera del rango 0x20 y 0x7E se muestran como un punto por no ser imprimibles. En el volcado del PSP aparecen únicamente dos caracteres, un espacio correspondiente al byte 20 y el símbolo ~ correspondiente al byte 7E.

Checkpoint 3 — Ensamblado y desensamblado
Captura: capturas/CP3_ensamblado_desensamblado.png
Observación. La instrucción ADD AX,BX se codificó como 01 D8 en lugar de 03 C3. Ambas codificaciones son válidas y equivalentes en el 8086, la diferencia está en cuál campo del byte ModRM actúa como destino. El desensamblado con U confirma que la instrucción resultante es la esperada y que ocupa los mismos dos bytes.

Checkpoint 4 — Modificación de memoria y direccionamiento directo
