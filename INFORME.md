## Primera parte: Estudiando el planificador de xv6-riscv

### 1) *¿Qué política de planificación utiliza `xv6-riscv` para elegir el próximo proceso a ejecutarse?*
 El sistema operativo **xv6-riscv** utiliza una política de **Round-Robin**(Por cada CPU) como planificador, esto debido a que se ejecuta un proceso durante un tiempo predeterminado (quantum). Cuando un proceso agota su quantum o se bloquea, el planificador elige el siguiente proceso de la cola de "RUNNABLE". 

Cuando un proceso se ejecuta cambia su estado a RUNNING, al terminar su ejecución, se espera que cambie su estado de nuevo a RUNNABLE o a otro estado más adecuado, permitiendo al planificador elegir y seleccionar otro proceso.

### 2) *¿Cuáles son los estados en los que un proceso puede permanecer en xv6-riscv y qué los hace cambiar de estado?*

Los estados en los que puede estar un proceso son:

* UNUSED
* USED
* SLEEPING
* RUNNABLE
* RUNNING
* ZOMBIE

Los cambios de estado ocurren por cuestiones como la espera de recursos(RUNNABLE -> SLEEPING), cuando finaliza la ejecución(RUNNING -> ZOMBIE), la reactivación después de un evento(SLEEPING -> RUNNABLE), cuando se acaba el tiempo del quantum (RUNNING -> RUNNABLE) o cuando el planificador elige un proceso (RUNNABLE -> RUNNING)

### 3) *¿Qué es un *quantum*? ¿Dónde se define en el código? ¿Cuánto dura un *quantum* en `xv6-riscv`?*

Un quantum es la cantidad de tiempo que un proceso tiene para ejecutarse en la CPU antes de ser interrumpido y ceder la CPU a otro proceso. Este tiempo es fijo para todos los procesos y define cuánto tiempo puede ejecutar cada uno antes de que el planificador decida cambiar de contexto a otro proceso.

Se define en el archivo **start.c** dentro de la función **timerinit()**, en la **línea 69**.

**Un quantum dura 1.000.000 ciclos de cpu(virtuales), aproximadamente 1/10 segundos en qemu.**

### 4) *¿En qué parte del código ocurre el cambio de contexto en `xv6-riscv`? ¿En qué funciones un proceso deja de ser ejecutado? ¿En qué funciones se elige el nuevo proceso a ejecutar?*
El cambio de contexto ocurre en la función **swtch**, que es llamada desde el **scheduler** y **sched**. Un proceso deja de ser ejecutado en las funciones **sched, yield y sleep**, que cambian el estado del proceso para cambiar a otro.

El nuevo proceso se elige en la función **scheduler**, que recorre la tabla de procesos buscando uno con estado **RUNNABLE**.

###  5)  *¿El cambio de contexto consume tiempo de un *quantum*?*
 El cambio de contexto no consume tiempo del quantum, ya que mientras un proceso se está ejecutando, este no realiza trabajo relacionado al cambio de contexto, solo realiza sus tareas. El cambio de contexto implica guardar el estado del proceso actual y restaurar el estado del nuevo, lo que consume tiempo de **CPU**.

## Segunda Parte: Medir operaciones de cómputo y de entrada/salida

### Experimento 1: ¿Cómo son planificados los programas iobound y cpubound?
Para este experimento utlizaremos el largo de quantum 10 veces más pequeño que el original y un valor de N constante con valor 4. Se realizaran mediciones en los 
siguientes escenarios:

a. iobench N &\
b. iobench N &; iobench N &; iobench N &\
c. cpubench N &\
d. cpubench N &; cpubench N &; cpubench N &\
e. iobench N &; cpubench N &; cpubench N &; cpubench N &\
f. cpubench N &; iobench N &; iobench N &; iobench N &

### 1) Describa los parámetros de los programas cpubench e iobench para este experimento (o sea, los define al principio y el valor de N. Tener en cuenta que podrían cambiar en experimentos futuros, pero que si lo hacen los resultados ya no serán comparables).

Paramétros de los programas iobench y cpubench:

* start_tick: tick en el cual comienza el proceso, contado desde que inició xv6 
* end_tick: tick en el cual finaliza el proceso, contado desde que inició xv6 
* elapsed_ticks: cantidad de ticks que demora en completarse el proceso (diferencia entre end_tick y start_tick)  
* total_cpu_kops: total de kilo operaciones que realiza la multiplicación de matrices
* total_iops: total de operaciones I/O
* metric: metrica a definir para realizar las mediciones. En nuestro caso la definimos como: metric = total_cpu_kops / elapsed_ticks. Análogamente para operaciones I/O: metric = total_iops / elapsed_ticks. Es decir, la cantidad de operaciones que se realizan por cada tick.

### 2) ¿Los procesos se ejecutan en paralelo? ¿En promedio, qué proceso o procesos se ejecutan primero? Hacer una observación cualitativa.

### 3) ¿Cambia el rendimiento de los procesos iobound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

### 4) ¿Cambia el rendimiento de los procesos cpubound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

### 5) ¿Es adecuado comparar la cantidad de operaciones de cpu con la cantidad de operaciones iobound?