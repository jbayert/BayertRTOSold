# BayertRTOS
This is a simple realtime operating system for the MSP430 microcontroller. 

## System Features
 * Preemptive operating system
 * Ability to create tasks on the fly.
 * All register data is saved and restored between tasks.
 * Seperate stacks for each task.
 * Memory management
 
## System Requirements 
The system code memory must be set to small and data memory must be set to small (so that the PC only stores 16-bits. Also you need to disable optimization. Also the tasks can not disable interrupts or use TimerA0.


## Getting Started
 Import the lib files into your project. And then in your main file use the import the RTOS.h file.
 ```c
 #include "RTOS.h"
 ```

## Public Functions

 * RTOS Setup

Sets up the RTOS system. This must be done only once before the system starts

```c
RTOSsetup(void);
```

 * RTOS init Task
 
RTOSinitTask sets up a task that will be run. This function adds a task to RTOS. This can be done while the RTOS is running, so a task can create another task. Returns -1 is it could not add a task because there was an error or the system already reaches the mask stacks. Otherwise returns a process id.

 ```c
 int process_id = RTOSinitTask(void (*pFun)(void));
 ```
 
 * RTOS Run

Runs all tasks. Once all tasks have finished the function returns 0 when all tasks finished without errors. 

```c
int errors = RTOSrun(void);
```

 * Killing a process 

Stops a process from running.

```c
killProcess(int proc_id);
```

 * Get a Process State

Returns the state of a process.

```c
char process_state = get_proc_state(int process)
```

Possible return values:

** PROCESS_NOT_ALLOCATED
process not defined yet
** PROCESS_NOT_STARTED
process has not started yet
** PROCESS_RUNNING
the process is running
** PROCESS_FINISHED_WITHOUT_ERRORS
process finished without any known errors

## Public Constants
These can all be found in the RTOS.h file and could be changed by the user.

*MAX_TASKS
the maximum number of tasks that will be called.
Default: 6 tasks
*STACK_SIZE
The maximum size (in words) of the stack of each function.
Note that the stack size must also contain enough space for the 16 registers to be stored between stacks.
Default: 40 words
*CYCLE_LENGTH
How often each task will be called (in microseconds).
Each task will run for CYCLE_LENGTH divided by (number of tasks)
Default: 40000 microseconds
*MAX_PROCESSES
How many process ids might be created in a single run
Default: 100


## Examples

There are two example projects. The first is a simple one that just implements a few tasks. The second uses the LCD screen and generates random numbers.

## How the RTOS is implemented 


Each task gets its own stack to use. Once a task time is up, the system will interrupt the task. The system will save the stack pointer and program counter, along with all the other registers. These values are saved onto the task’s stacks when other tasks are running. A typical stack for a task will look like this:

![Image of a Typical Stack](./doc/TypicalStack.jpg)

When a task finishes it will return from the bottom of the stack. This return value is saved in the last value of the stack. The stack will be deallocated and can be reused after a task has finished.

*Process ID
Each task receives a unique Id. This id can be used to get the current status of the task or to kill the task. The process ids are not reused unless there are more than MAX_PROCESSES have been created.

*Maximum run time
Every function will be guaranteed to be called every CYCLE_LENGTH microseconds. The run time of each task is dynamic, so that each task will run CYCLE_LENGTH divided by the (# of tasks). 

*TimerA0
The timerA0 interrupt is used to schedule the change of each task. This timer is set up to use the TA0CCTL0 interrupt.


