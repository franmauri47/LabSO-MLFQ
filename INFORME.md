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
