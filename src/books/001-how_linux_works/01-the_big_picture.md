# Chapter 1 - The Big Picture

## 1.1 Levels & Layers of Abstraction in a Linux System

Why we should use ***abstraction***:

* The most effective way to understand how system work
* Ignore the details
* Concentrate on basic purpose and operation

Some terms of abstraction in computer software:

* Subsystem
* Module
* Component
* Package

| User Processes      | Linux Kernel                                                 | Hardware                 |
| ------------------- | ------------------------------------------------------------ | ------------------------ |
| GUI, Servers, Shell | Process Management, System Calls, Memory Management, Device Driver | CPU, RAM, Disks, Network |
| User space          | kernel space: The memory area that only kernel has access to it |                          |

## 1.2 Hardware: Understanding Main Memory

**Main Memory:** Most important part

**CPU:** An operator on memory, it reads data from memory and write it back.

**State (in memory):** A particular arrangement of bits.

**Image (in memory):** A particular physical arrangement of bits.

## 1.3 The Kernel

1. **Processes:** Which process is allow to use CPU.
2. **Memory:** Keep track of all memory
   * What is currently allocated to particular process
   * What might to be shared between process
   * What is free
3. **Device Drivers:** Interface between Hardware and process
4. **System Calls & Support:**  Processes interface to communicate with the kernel.

### 1.3.1 Process Management

1\. starting 2.Pausing 3. Resuming 4. Scheduling 5. Terminating

Context Switching: The act of one process giving up control of CPU to another process

Time Slice: Each piece of  time gives a process enough time for significant computation

> [!NOTE]
>
> The kernel is responsible for context switching.

::question: When kernel runs? It runs between process time slices during a context switch

### 1.3.2 Memory Management

Managing memory during a context switching (Conditions):

* Kernel's specific area in memory
* Particular section area in memory for each user processes.
* Division without access to each other area in ram for all use processes
