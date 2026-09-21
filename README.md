# Laboratorio Unidad 3 — Manejo del DEBUG
Este repositorio contiene el desarrollo de las dos partes del laboratorio guiado de manejo del DEBUG en DOSBox. La Parte 1 explora el estado inicial del procesador, la inspección de memoria con los comandos R, F, D y U, y la modificación puntual con E verificada mediante direccionamiento directo a memoria. La Parte 2 ensambla programas con el comando A, los ejecuta instrucción a instrucción con T y analiza el mecanismo de bucle LOOP frente a un bucle equivalente construido con DEC y JNZ.

Entorno utilizado

DOSBox 0.74-3 sobre Windows. El segmento asignado durante las sesiones fue 0724, por lo que las direcciones mostradas en las capturas aparecen como 0724:XXXX en lugar del 1357:XXXX del enunciado. El valor del segmento lo asigna el sistema en cada sesión.

Los directorios de trabajo se nombraron LAB3POS1 y LAB3POS2 porque el sistema de archivos del DOS limita los nombres a ocho caracteres y LAB3POST1 y LAB3POST2 se truncaban ambos al mismo nombre LAB3POST.

# PARTE 1 — Exploración con DEBUG
## Checkpoint 1 — Estado de registros
Captura: capturas/CP1_registros.png

Al iniciar DEBUG sin archivo, los cuatro registros de propósito general aparecen en cero, SP apunta a FFFE y los cuatro registros de segmento apuntan al mismo párrafo de memoria. El IP inicia en 0100, que es la primera dirección ejecutable después del PSP.

Al modificar AX con R AX y el valor 1234, la siguiente invocación de R muestra únicamente ese registro con el valor nuevo y el resto sin cambio, lo que confirma que la modificación es selectiva.

## Checkpoint 2 — Volcado hexadecimal
Captura: capturas/CP2_volcado_memoria.png
Con F 200 L40 AB CD EF se rellenaron 64 bytes y el volcado con D 200 L40 muestra el patrón AB CD EF repetido en las cuatro filas.
Al observar la salida del comando D se identifican tres columnas. La primera columna corresponde a la dirección donde está ubicada esa línea en la memoria, esta se escribe en dos partes separada por dos puntos. El 0724 corresponde al segmento y el 0200 es el desplazamiento. La segunda columna es el contenido real de la memoria, compuesto por 16 bytes, donde cada uno son dos dígitos hexadecimales. La tercera columna es la interpretación ASCII, los valores que se encuentran fuera del rango 0x20 y 0x7E se muestran como un punto por no ser imprimibles. En el volcado del PSP aparecen únicamente dos caracteres, un espacio correspondiente al byte 20 y el símbolo ~ correspondiente al byte 7E.

## Checkpoint 3 — Ensamblado y desensamblado
Captura: capturas/CP3_ensamblado_desensamblado.png
Observación. La instrucción ADD AX,BX se codificó como 01 D8 en lugar de 03 C3. Ambas codificaciones son válidas y equivalentes en el 8086, la diferencia está en cuál campo del byte ModRM actúa como destino. El desensamblado con U confirma que la instrucción resultante es la esperada y que ocupa los mismos dos bytes.

## Checkpoint 4 — Modificación de memoria y direccionamiento directo
Captura: capturas/CP4_memoria_direccionamiento.png
La secuencia F 300 L10 00 dejó la región en ceros, E 300 78 56 escribió dos bytes puntuales y el volcado posterior confirmó que los catorce bytes restantes permanecieron sin cambio. Leídos en little-endian, los dos bytes escritos representan el valor 5678.

La instrucción MOV AX,[0300] se ensambló en 0320 para no sobrescribir el programa del paso anterior y se codificó como A1 00 03. Tras restablecer el IP a 0320 y ejecutar T, el registro AX quedó en 5678, lo que confirma la lectura del valor previamente escrito con E.
 
## Decisión Técnica — Verificación No Destructiva de una Escritura en Memoria
El comando correcto para confirmar la escritura del Paso 11 es D. Este comando lee la memoria y la imprime sin alterarla, lo que resulta adecuado porque una verificación que cambie el estado dejaría de verificar el estado original. El comando E no es una opción porque su función es escribir y al invocarlo sin la lista de bytes entra en modo interactivo, donde cualquier tecla presionada por error sobrescribiría el valor que se intenta comprobar. El comando F opera sobre un rango igual que D, pero su función es rellenar repitiendo un patrón, por lo que destruiría el contenido que se desea verificar. El comando R tampoco es una opción, porque actúa sobre los registros del procesador y no sobre la memoria, de modo que no puede observar la dirección 0300, y además con un argumento permite modificar el registro indicado. 

## Decisión Técnica — Modo de Direccionamiento Inmediato vs. Directo a Memoria
Al comparar el direccionamiento inmediato con el direccionamiento directo a memoria, aunque ambas ocupen 3 bytes la que requiere un ciclo adicional de acceso al bus de memoria es la que va directo a memoria. Con el direccionamiento inmediato el dato ya está en la instrucción. En cambio con la segunda, viaja es la dirección, por lo cual el procesador debe hacer un segundo acceso al bus para obtener el dato.
Es preferible el direccionamiento directo cuando el dato puede cambiar en la ejecución del programa ya que con el inmediato el valor queda fijo en el momento de ensamblar. Con el directo, si el dato cambia, como el valor escrito con E en el Paso 11, no se debe reescribir la instrucción.
El comando U permite confirmarlo porque traduce los bytes almacenados sin ejecutar ninguna instrucción. En la salida se observa que la primera codificación es B8 05 00 y se desensambla como MOV AX,0005, mientras que la segunda es A1 00 03 y se desensambla como MOV AX,[0300]. El opcode distinto y los corchetes en el operando identifican cada modo de direccionamiento.


# PARTE 2 — Ensamblado y ejecución paso a paso

## Checkpoint 1 — Traza del programa de suma
Captura: capturas/CP1_traza_suma.png

| Instrucción | AX | BX | CX | IP siguiente | ZF | CF | SF |
|---|---|---|---|---|---|---|---|
| MOV AX, 000A |000A|0000|0000|0103|NZ|NC|PL|
| MOV BX, 0005 |000A|0005|0000|0106|NZ|NC|PL|
| MOV CX, 0003 |000A|0005|0003|0109|NZ|NC|PL|
| ADD AX, BX |000F|0005|0003|010B|NZ|NC|PL|
| ADD AX, CX |0012|0005|0003|010D|NZ|NC|PL|

Al finalizar las dos instrucciones ADD el registro AX quedó en 0012, que equivale a 18 en decimal. Las banderas se mantuvieron en NZ, NC y PL porque ningún resultado fue cero, negativo ni generó acarreo.

## Checkpoint 2 — Traza del bucle con LOOP
Capturas: capturas/CP2_traza_loop.png y capturas/CP2_traza_loop_2.png

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿LOOP salta? |
|---|---|---|---|---|---|
| — | MOV AX, 0000 |0000|0003|0103| — |
| — | MOV CX, 0004 |0000|0004|0106| — |
| 1 | ADD AX, 0002 |0002|0004|0109| — |
| 1 | LOOP 0106 |0002|0003|0106|Sí|
| 2 | ADD AX, 0002 |0004|0003|0109| — |
| 2 | LOOP 0106 |0004|0002|0106|Sí|
| 3 | ADD AX, 0002 |0006|0002|0109| — |
| 3 | LOOP 0106 |0006|0001|0106|Sí|
| 4 | ADD AX, 0002 |0008|0001|0109| — |
| 4 | LOOP 0106 |0008|0000|010B|No|
| — | INT 20 |0008|0000| — | — |

Observaciones. La traza se dividió en dos capturas porque la pantalla del DOS es de veinticinco líneas. En la primera fila CX aparece en 0003 por residuo de la sesión anterior y solo toma el valor 0004 tras la segunda instrucción. BX permanece en 0005 durante todo el recorrido porque este programa nunca lo modifica. Mientras el bucle está activo el IP alterna entre 0109 y 0106, y cuando CX llega a cero continúa hacia 010B en lugar de regresar, lo que evidencia que LOOP dejó de saltar. El resultado final es AX igual a 0008.

## Análisis del Código Máquina con D
El volcado con D sobre la región de código muestra los trece bytes que componen el programa del bucle.

| Instrucción | Bytes | Tamaño |
|---|---|---|
| MOV AX,0000 | B8 00 00 | 3 |
| MOV CX,0004 | B9 04 00 | 3 |
| ADD AX,+02 | 83 C0 02 | 3 |
| LOOPW 0106 | E2 FB | 2 |
| INT 20 | CD 20 | 2 |

El byte E2 es el opcode de LOOP y FB es el desplazamiento relativo con signo. El valor corresponde a menos cinco en complemento a dos, calculado desde la dirección siguiente al LOOP, que es 010B, hasta el inicio del cuerpo del bucle en 0106.

## Decisión Técnica — Selección de Mecanismo de Control de Bucle (LOOP vs. DEC/JNZ)
Al comparar ambos mecanismos de control de bucle en el caso de la práctica, el mecanismo que se recomienda es el LOOP, que ocupa 2 bytes frente a los 3 del DEC/JNZ. Por cada iteración el procesador extraería 2 instrucciones en vez de 3, lo cual al calcular el total de instrucciones ejecutadas da 11 frente a las 15 del otro mecanismo.

Se opta por DEC/JNZ cuando el cuerpo del bucle necesita el registro CX para otra operación, ya que LOOP solo funciona con CX mientras que DEC y JNZ sirven con cualquier registro. También se prefiere cuando la condición de salida no es que el contador llegue a cero sino el resultado de una comparación.

Se verifica con el comando U al observar las direcciones de cada línea del desensamblado, que muestran cuántos bytes ocupa cada instrucción. Para el primer mecanismo el LOOP está en 0109 y el INT 20 en 010B, o sea 2 bytes. Para el segundo, DEC está en 0209 y el INT 20 en 020C, o sea 3 bytes para DEC y JNZ juntos.

## Checkpoint 3 — Traza del bucle con DEC y JNZ
Capturas: capturas/CP3_traza_dec_jnz.png a CP3_traza_dec_jnz_3.png y capturas/CP3_demostracion_g.png. La traza se dividió en varias capturas porque la pantalla del DOS es de veinticinco líneas y la secuencia completa no cabe en una sola.

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿JNZ salta? |
|---|---|---|---|---|---|
| — | MOV AX, 0000 | 0000 | 0000 | 0203 | — |
| — | MOV CX, 0004 | 0000 | 0004 | 0206 | — |
| 1 | ADD AX, 0002 | 0002 | 0004 | 0209 | — |
| 1 | DEC CX | 0002 | 0003 | 020A | — |
| 1 | JNZ 0206 | 0002 | 0003 | 0206 | Sí |
| 2 | ADD AX, 0002 | 0004 | 0003 | 0209 | — |
| 2 | DEC CX | 0004 | 0002 | 020A | — |
| 2 | JNZ 0206 | 0004 | 0002 | 0206 | Sí |
| 3 | ADD AX, 0002 | 0006 | 0002 | 0209 | — |
| 3 | DEC CX | 0006 | 0001 | 020A | — |
| 3 | JNZ 0206 | 0006 | 0001 | 0206 | Sí |
| 4 | ADD AX, 0002 | 0008 | 0001 | 0209 | — |
| 4 | DEC CX | 0008 | 0000 | 020A | — |
| 4 | JNZ 0206 | 0008 | 0000 | 020C | No |
| — | INT 20 | 0008 | 0000 | — | — |

Observaciones. En la última ejecución de DEC las banderas pasan de NZ a ZR, que es el momento exacto en que CX llega a cero y provoca que el JNZ siguiente no salte. Este comportamiento no se observa con LOOP, porque LOOP consulta el valor de CX de forma directa y no depende de las banderas. El indicador de acarreo permanece en NC durante todo el recorrido, lo que confirma que DEC afecta la bandera de cero pero no la de acarreo.

| Mecanismo | Bytes de control | Ejecuciones de T |
|---|---|---|
| LOOP | 2 | 11 |
| DEC y JNZ | 3 | 15 |

Ambas versiones producen el mismo resultado final, AX igual a 0008. La diferencia está en el costo, cuatro instrucciones adicionales en total, una por cada iteración, correspondientes a la instrucción de control extra que DEC y JNZ necesitan frente a la instrucción única de LOOP.

El desplazamiento de JNZ 0206 se codifica como FA, que corresponde a menos seis, calculado desde 020C hasta 0206 con el mismo criterio del LOOP.

## Demostración con el comando G
Tras restablecer el IP a 0200, el comando G 20C ejecutó el programa completo a velocidad normal y se detuvo en la dirección indicada, mostrando AX igual a 0008 sin desplegar ninguno de los estados intermedios.

## Decisión Técnica — Comando de Verificación para Bucles de Muchas Iteraciones (T vs. G)
Para verificar el resultado de 100 iteraciones usar el comando G resulta más práctico porque con un solo comando se obtiene el mismo resultado que escribiendo cientos de veces el comando T. Sin embargo, se pierde la traza de todos los estados intermedios, teniendo menor control de cómo cambian AX y CX en cada vuelta, al igual que de las banderas y el recorrido del IP.

Usar el comando T fue obligatorio en los Pasos 4, 7 y 11 porque las tablas de traza pedían el valor de los registros después de cada instrucción y G se salta todos esos pasos.

Para confirmar solo AX después de G se escribe R AX y se presiona Enter sin ingresar ningún valor, de esta forma no se modifica AX y se verifica su valor final.

## Conclusiones
Finalizada esta práctica se logró aprender y afianzar el uso de cada comando del DEBUG, así como comprender mejor los registros generales, los registros de segmento y las banderas del procesador, lo que permite tener un mejor control de cómo se ejecutan las instrucciones a bajo nivel. Se observó que una misma instrucción puede tener varias codificaciones válidas, como ADD AX,BX que se codificó como 01 D8 en lugar de 03 C3 y ADD AX,0002 que se codificó como 83 C0 02 en lugar de 05 02 00, y que el comando U permite identificarlas sin ejecutar nada. También se comprobó que el procesador ejecuta lo que apunte CS:IP sin distinguir entre código y datos, lo cual se vio al desensamblar la memoria sin inicializar. Además, se evidenció que LOOP es más compacto mientras que DEC/JNZ es más flexible. Por último, T y G resultan útiles según el caso, T cuando se necesita observar cada paso y G cuando solo interesa el resultado final.
