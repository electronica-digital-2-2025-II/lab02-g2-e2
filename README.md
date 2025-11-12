[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/sEFmt2_p)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=21496800&assignment_repo_type=AssignmentRepo)
# Lab02 - Unidad Aritmético-Lógica.

# Integrantes
Juan Mateo jimenez
Pablo Cesar Rincon
# Informe

Indice:

1. [Diseño implementado](#diseño-implementado)
2. [Simulaciones](#simulaciones)
3. [Implementación](#implementación)
4. [Conclusiones](#conclusiones)
5. [Referencias](#referencias)

## Diseño implementado


El presente proyecto consiste en el diseño e implementación de una **Unidad Aritmético-Lógica (ALU)** de **4 bits** desarrollada en **Verilog HDL** y destinada a operar dentro de una **FPGA Zybo Z7**.  
Esta ALU puede ejecutar operaciones aritméticas y lógicas fundamentales mediante una señal de control (`Sel`) de 3 bits, que define cuál de las funciones internas se activa.  

La ALU fue diseñada bajo un enfoque **modular**, integrando submódulos independientes para cada operación: **suma/resta, multiplicación, corrimiento, movimiento de bits y operación lógica OR**.  
Cada uno de estos módulos produce un resultado que se enruta al bus de salida general (`Y`), controlado por un bloque `case` dentro del módulo principal.  
La arquitectura se complementa con la detección de **banderas de estado** (`Over` y `Zero`) y la coordinación de señales secuenciales (`clk`, `start`, `mult_done`) para garantizar un flujo de operación estable.

---

### Estructura del diseño

El módulo principal, `alu.v`, recibe los siguientes puertos:

| Señal | Tipo | Tamaño | Descripción |
|--------|------|---------|-------------|
| `A`, `B` | Entrada | 4 bits | Operandos principales. |
| `Sel` | Entrada | 3 bits | Selección de la operación. |
| `clk` | Entrada | 1 bit | Señal de reloj. |
| `rst` | Entrada | 1 bit | Reinicio del sistema. |
| `start` | Entrada | 1 bit | Pulso de inicio para operaciones secuenciales (multiplicación). |
| `mult_done` | Entrada | 1 bit | Señal de finalización proveniente del módulo de multiplicación. |
| `Y` | Salida | 8 bits | Resultado general de la operación. |
| `Over` | Salida | 1 bit | Bandera de desbordamiento aritmético. |
| `Zero` | Salida | 1 bit | Bandera que indica resultado igual a cero. |

Internamente, la ALU combina bloques combinacionales y secuenciales.  
Los módulos aritméticos (`sum_res4b`, `mult4b`) y los lógicos (`Cor`, `Move`) se integran mediante un sistema de **comparadores (`comp8b`)** que activan condiciones de control específicas dependiendo del valor de `Sel`.

---

### Operaciones soportadas

| Código `Sel` | Operación | Módulo | Descripción |
|---------------|------------|---------|--------------|
| `000` | **Suma** | `sum_res4b` | Realiza la suma de `A` y `B` con control de *carry* y detección de `overflow`. |
| `001` | **Resta** | `sum_res4b` | Ejecuta la resta binaria `A - B`, reutilizando el sumador con inversión de bits y *carry in*. |
| `010` | **Multiplicación** | `mult4b` | Multiplicador secuencial de 4×4 bits controlado por `clk` y `start`. |
| `011` | **Movimiento/Corrimiento** | `Move.v` | Desplazamiento lógico del operando `A` controlado internamente. |
| `100` | **Operación lógica OR** | `Cor.v` | Realiza la operación lógica bit a bit OR entre `A` y `B`. |

La lógica de selección de salida se define en un bloque `always @(*)`:
<p align="center">
  <img src="images/Captura de pantalla 2025-11-11 202651.png" alt="parte del codigo" width="450">
</p>




## Descripción

La **Unidad Aritmético-Lógica (ALU)** desarrollada en este laboratorio se diseñó con un enfoque **modular jerárquico**, permitiendo integrar múltiples operaciones aritméticas y lógicas dentro de un único bloque controlado mediante un código de selección (`Sel`).  
El objetivo principal fue construir un sistema capaz de ejecutar **cinco operaciones diferentes** (suma, resta, multiplicación, corrimiento y OR) sobre dos operandos de 4 bits, mostrando el resultado de 8 bits en los LEDs de la tarjeta FPGA.

---

### Arquitectura general del sistema

El sistema completo se compone de dos niveles jerárquicos:

1. **Nivel lógico-funcional (`alu.v`)**  
   En este nivel se agrupan los módulos que ejecutan las operaciones básicas:
   - `sum_res4b`: sumador/restador de 4 bits.  
   - `mult4b`: multiplicador secuencial 4×4 bits.  
   - `Move`: operación de corrimiento o desplazamiento lógico.  
   - `Cor`: operación lógica OR bit a bit.  
   - `Comp8b`: comparador de igualdad utilizado para activar banderas de control.

   La ALU utiliza estos módulos como componentes internos, conectándolos a través de un **bus de control de 3 bits (`Sel`)** que define qué operación se selecciona en cada momento.  
   El resultado de cada módulo se enruta al bus de salida general `Y`, mientras que el cálculo de banderas (`Over` y `Zero`) se realiza mediante comparadores y operaciones lógicas internas.

2. **Nivel físico de interacción (`top_alu.v`)**  
   Este módulo actúa como interfaz entre la lógica digital y los periféricos de la FPGA Zybo Z7.  
   Permite que el usuario controle las operaciones mediante **switches y botones**, y observe los resultados a través de los **LEDs**.

   En este bloque se implementa el **almacenamiento sincronizado** de los operandos `A` y `B`, así como la selección `Sel`, mediante un **botón de start** (`start_btn`).  
   De esta forma, los valores de entrada solo cambian cuando el usuario presiona el botón, evitando alteraciones durante la ejecución de operaciones secuenciales (como la multiplicación).

---

### Flujo de funcionamiento

1. El usuario configura los operandos `A` y `B` mediante los **switches** de la FPGA, y selecciona la operación deseada con los **3 switches de control (`Sel`)**.
2. Al presionar el botón **START**, el módulo `top_alu` transfiere los valores de `swA`, `swB` y `swSel` hacia los registros internos `A_reg`, `B_reg` y `Sel_reg`.  
   Esto garantiza estabilidad en los datos durante todo el proceso.
3. La señal `start_reg` se propaga hacia la ALU y, dependiendo del valor de `Sel`, puede activar el inicio de una operación secuencial, como la multiplicación (`mult_start`).
4. Dentro del módulo `alu`, se ejecuta la operación correspondiente:
   - Para **suma o resta**, el resultado se calcula instantáneamente usando el módulo `sum_res4b`.  
   - Para **multiplicación**, se inicia una secuencia controlada por reloj dentro de `mult4b`, que al finalizar activa la señal `mult_done`.  
   - Para **corrimiento** y **OR**, las salidas son combinacionales y se generan de inmediato.  
5. Finalmente, el resultado `Y` se envía a los **8 LEDs principales**, mientras que los LEDs auxiliares indican el estado de las banderas:
   - `ledOver`: se enciende cuando ocurre un desbordamiento.  
   - `ledZero`: se enciende cuando el resultado es igual a cero.

---

### Códigos de operación (opcode)

| `Sel` | Operación ejecutada | Descripción |
|--------|----------------------|--------------|
| `000` | Suma (`A + B`) | Ejecuta la suma binaria con detección de *carry* y *overflow*. |
| `001` | Resta (`A - B`) | Realiza la resta binaria con detección de signo y desbordamiento. |
| `010` | Multiplicación (`A × B`) | Opera de forma secuencial bajo control de reloj y `start`. |
| `011` | Corrimiento lógico | Desplaza los bits del operando `A` según lo definido en el módulo `Move.v`. |
| `100` | OR lógico (`A OR B`) | Calcula la operación lógica bit a bit entre ambos operandos. |

---

### Señales de control y estado

- **`clk`**: sincroniza las operaciones secuenciales, principalmente la multiplicación.  
- **`rst_btn`**: reinicia todos los registros y estados del sistema.  
- **`start_btn`**: registra los valores de entrada y activa el inicio de ejecución.  
- **`Over`**: bandera de desbordamiento en suma/resta.  
- **`Zero`**: bandera activa cuando el resultado `Y` es cero.  
- **`mult_done`**: señal interna que indica que el multiplicador ha completado su secuencia.

---

### Comportamiento esperado

- Las operaciones **combinacionales** (suma, resta, OR y corrimiento) entregan resultados **instantáneos**, visibles inmediatamente en los LEDs.  
- La **multiplicación** presenta un retardo proporcional al número de ciclos de reloj necesarios para completar la secuencia, tras lo cual se actualiza el resultado y se activa la bandera `mult_done`.  
- Las banderas `Over` y `Zero` se actualizan automáticamente según el valor de salida `Y`.

---

En resumen, la ALU implementada combina la lógica combinacional y secuencial de manera eficiente, utilizando una estructura modular controlada por un único bus de selección.  
El uso del botón de inicio permite un control estable y reproducible sobre las operaciones, facilitando tanto la simulación como la verificación experimental en la FPGA.





### Diagrama

## Simulaciones 
<p align="center">
  <img src=".github/resta.png" alt="Montaje del laboratorio digital" width="450">
</p>


<p align="center">
  <img src=".github/simulacion alu.png" alt="Montaje del laboratorio digital" width="450">
</p>




## Implementación

## Conclusiones

## Referencias
