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
Captura: capturas/CP4_memoria_direccionamiento.png
La secuencia F 300 L10 00 dejó la región en ceros, E 300 78 56 escribió dos bytes puntuales y el volcado posterior confirmó que los catorce bytes restantes permanecieron sin cambio. Leídos en little-endian, los dos bytes escritos representan el valor 5678.

La instrucción MOV AX,[0300] se ensambló en 0320 para no sobrescribir el programa del paso anterior y se codificó como A1 00 03. Tras restablecer el IP a 0320 y ejecutar T, el registro AX quedó en 5678, lo que confirma la lectura del valor previamente escrito con E.
 
Decisión Técnica — Verificación No Destructiva de una Escritura en Memoria
El comando correcto para confirmar la escritura del Paso 11 es D. Este comando lee la memoria y la imprime sin alterarla, lo que resulta adecuado porque una verificación que cambie el estado dejaría de verificar el estado original. El comando E no una opción porque su función es escribir y al invocarlo sin la lista de bytes entra en modo interactivo, donde cualquier tecla presionada por error sobrescribiría el valor que se intenta comprobar. El comando F opera sobre un rango igual que D, pero su función es rellenar repitiendo un patrón, por lo que destruiría el contenido que se desea verificar. El comando R tampoco es una opción, porque actúa sobre los registros del procesador y no sobre la memoria, de modo que no puede observar la dirección 0300, y además con un argumento permite modificar el registro indicado. 

Decisión Técnica — Modo de Direccionamiento Inmediato vs. Directo a Memoria
Al comparar el direccionamiento inmediato con el direccionamiento directo a memoria, aunque ambas ocupen 3 bytes la que requiere un ciclo adicional de acceso al bus de memoria es la que va directo a memoria. Con el direccionamiento inmediato el dato ya esta en la instruccion. En cambio con la segunda, viaja es la dirección, por lo cual el procesador debe hacer un segundo acceso al bus para obtener el dato.
Es preferible el direccionamiento directo cuando el dato puede cambiar en la ejecución del programa ya que con el inmediato el valor queda fijo en el momento de ensamblar, asi si se debe cambiar el dato no se debe reescribir la instrucción.
El comando U permite confirmarlo porque traduce los bytes almacenados sin ejecutar ninguna instrucción. En la salida se observa que la primera codificación es B8 05 00 y se desensambla como MOV AX,0005, mientras que la segunda es A1 00 03 y se desensambla como MOV AX,[0300]. El opcode distinto y los corchetes en el operando identifican cada modo de direccionamiento.


PARTE 2 — Ensamblado y ejecución paso a paso
Checkpoint 1 — Traza del programa de suma

Checkpoint 2 — Traza del bucle con LOOP

Capturas: capturas/CP2_traza_loop.png y capturas/CP2_traza_loop_2.png

Análisis del Código Máquina con D

Decisión Técnica — Selección de Mecanismo de Control de Bucle (LOOP vs. DEC/JNZ)

Checkpoint 3 — Traza del bucle con DEC y JNZ

Demostración con el comando G

Decisión Técnica — Comando de Verificación para Bucles de Muchas Iteraciones (T vs. G)

Conclusiones
