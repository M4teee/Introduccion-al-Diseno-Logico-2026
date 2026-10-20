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
- `simulation/*.vwf`: vectores de onda para la simulación, con archivos diferenciados por operación y pruebas integradas.

## Apertura y compilación

1. Abrir `TPI.qpf` con Quartus II 13.1 o una versión compatible.
2. Confirmar que `TPI` figure como entidad de nivel superior y que los archivos `.bdf` referenciados estén en el directorio del proyecto.
3. Ejecutar **Processing > Start Compilation**.
4. Para inspeccionar los estímulos, abrir los archivos `.vwf` de `simulation/` en el Waveform Editor. La simulación requiere una versión compatible de ModelSim-Altera y la configuración EDA correspondiente.

## Estructura del proyecto

- `TPI.qpf`, `TPI.qsf`, `TPI.bdf`: proyecto Quartus, asignaciones y esquema de nivel superior.
- `*.bdf`, `*.bsf`: esquemas y símbolos de los bloques; se mantienen junto al proyecto para conservar las referencias jerárquicas de Quartus.
- `simulation/*.vwf`: vectores de onda asociados al proyecto.
- `.gitignore`: excluye archivos temporales y resultados generados por Quartus.
