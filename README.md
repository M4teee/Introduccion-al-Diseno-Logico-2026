# ALU de 4 bits — Introducción al Diseño Lógico

Unidad aritmético-lógica secuencial implementada con esquemas de bloques en Quartus II. El diseño opera sobre un acumulador de 4 bits (`A`) y un registro auxiliar de 4 bits (`B`), selecciona una de ocho operaciones mediante `CodOp[2..0]` y mantiene las flags de estado `V`, `N`, `Z` y `C`.

## Especificaciones

- Operandos y resultado: 4 bits, interpretados en complemento a dos para operaciones aritméticas.
- Entradas del diseño superior: `A[3..0]`, `B[3..0]`, `LOADA`, `LOADB`, `CLK` y `CodOp[2..0]`.
- Salidas: `A_out[3..0]`, `B_out[3..0]` y las flags `V`, `N`, `Z`, `C`.
- Los registros A, B y Estado son sincrónicos. Las cargas de A y B se controlan con `LOADA` y `LOADB`; las operaciones actualizan A y/o flags según el opcode.

## Operaciones

| `CodOp` | Operación | Comportamiento |
| --- | --- | --- |
| `000` | NOP | Conserva los registros y las flags. |
| `001` | ADD | Suma A + B y escribe el resultado en A. |
| `010` | SUB | Calcula A − B y escribe el resultado en A. |
| `011` | CMP | Calcula A − B para actualizar las flags, sin escribir A. |
| `100` | NOT | Invierte cada bit de A y escribe el resultado en A. |
| `101` | AND | Calcula A AND B bit a bit y escribe el resultado en A. |
| `110` | OR | Calcula A OR B bit a bit y escribe el resultado en A. |
| `111` | ASR | Desplaza A una posición a la derecha con extensión de signo y escribe en A. |

ADD, SUB y CMP actualizan las cuatro flags. NOT, AND, OR y ASR actualizan `N` y `Z`, preservando `V` y `C`. NOP conserva el estado completo. `N` corresponde al bit más significativo del resultado y `Z` indica resultado cero. En operaciones aritméticas, `C` es el acarreo de salida; en resta y comparación, `C=1` indica `A ≥ B`. `V` representa desbordamiento en complemento a dos.

## Arquitectura

El decodificador 3 a 8 genera señales mutuamente excluyentes para las operaciones. ADD, SUB y CMP comparten un sumador de 4 bits con acarreo serie; para SUB y CMP se complementa B y se activa el acarreo de entrada, implementando la resta en complemento a dos. El bloque XOR condicional se reutiliza también para NOT mediante selección de su operando.

Los resultados de las operaciones que escriben A convergen en un bus común mediante buffers tri-state habilitados por el decodificador. CMP calcula flags sin habilitar la escritura del acumulador. El registro A incorpora la selección necesaria para carga paralela, escritura del resultado y desplazamiento aritmético a derecha. El registro Estado almacena las cuatro flags con habilitaciones de carga independientes para conservar aquellas que una operación no modifica.

## Herramientas

- Intel Quartus II 13.1, versión consignada en `TPI.qpf` y `TPI.qsf`.
- Block Diagram/Schematic Files (`.bdf`) para los esquemas y símbolos de bloque (`.bsf`).
- Vector Waveform Files (`.vwf`) para los estímulos de simulación.
- El proyecto configura ModelSim-Altera (Verilog) como herramienta EDA de simulación. No se incluyen fuentes Verilog/VHDL del diseño.

## Archivos del proyecto

- `TPI.qpf`, `TPI.qsf`: proyecto, revisión y asignaciones de Quartus; `TPI.bdf` es la entidad de nivel superior (`TPI`).
- `TPI.bdf`: interconexión de registros, decodificador, bloques de operación y bus de resultados.
- `full-adder.bdf`, `sumador4.bdf`: sumador completo y sumador de 4 bits.
- `decoder3to8.bdf`: decodificador de opcode.
- `and-arr4.bdf`, `or-arr4.bdf`, `xor-arr4.bdf`: arreglos lógicos de 4 bits.
- `registro-A.bdf`, `registro-B.bdf`, `registro-estado.bdf`: registros de datos y flags.
- `V-flag-logic.bdf`, `Z-flag-logic.bdf`: lógica asociada a las flags de overflow y cero.
- `*.bsf`: símbolos de los componentes utilizados por los esquemas.
- `*.vwf`: vectores de onda para la simulación, con archivos diferenciados por operación y pruebas integradas.

## Apertura y compilación

1. Abrir `TPI.qpf` con Quartus II 13.1 o una versión compatible.
2. Confirmar que `TPI` figure como entidad de nivel superior y que los archivos `.bdf` referenciados estén en el directorio del proyecto.
3. Ejecutar **Processing > Start Compilation**.
4. Para inspeccionar los estímulos, abrir los archivos `.vwf` en el Waveform Editor. La simulación requiere una versión compatible de ModelSim-Altera y la configuración EDA correspondiente.

El archivo `TPI.qsf` contiene una ruta absoluta de salida de simulación de Windows (`C:/Users/ALAN/...`) que pertenece al entorno original. Puede ser necesario actualizar la ruta EDA de simulación en Quartus antes de ejecutar ModelSim en otro equipo. Los archivos del repositorio incluyen los proyectos y vectores, pero no resultados de compilación generados.

## Verificación documentada

El informe del trabajo práctico describe una simulación con carga inicial `A=0101`, `B=0011` y recorrido secuencial de los ocho códigos, verificando la actualización y retención de estado entre flancos de reloj. Los resultados informados son:

| `CodOp` | Operación | A después de la operación | V | N | Z | C |
| --- | --- | --- | --- | --- | --- | --- |
| `000` | NOP | `0101` | — | — | — | — |
| `001` | ADD | `1000` | 1 | 1 | 0 | 0 |
| `010` | SUB | `0101` | 1 | 0 | 0 | 1 |
| `011` | CMP | `0101` | 0 | 0 | 0 | 1 |
| `100` | NOT | `1010` | 0 | 1 | 0 | 1 |
| `101` | AND | `0010` | 0 | 0 | 0 | 1 |
| `110` | OR | `0011` | 0 | 0 | 0 | 1 |
| `111` | ASR | `0001` | 0 | 0 | 0 | 1 |

En NOP, las flags se conservan; por eso se muestran con guiones. El informe no incluye reportes de compilación de Quartus ni artefactos de simulación exportados.

## Equipo

Trabajo Integrador de Introducción al Diseño Lógico (E1301), Grupo 11:

- Lucio Agustin Persano
- Elias Brea
- Mateo Gau
- Alan Tolaba
- Mateo Rivas Aramburu

