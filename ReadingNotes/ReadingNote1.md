
Operating System: Reading Notes for Week1 
===
   * Chapter 1.1

      * two main function of OS: extending the machine & managing resources
      * extending machine

         * Hardware control are complex, so OS present the user with the equivalent of an extended machine or virtual machine that is easier to program than the underlying hardware

      * resource manager

         * Time multiplexing(share)

            * Determining how the resource is time multiplexed-who goes next and for how long—is the task of the operating system

         * Space multiplexing(share)

            * Allocating disk space and keeping track of who is using which disk hocks is a typical operating system resource management task



   * Chapter 1.3 The operating system zoo

      * Mainframe operating systems

         * Distinguishing themselves from personal OS in term of their I/O capacity
         * Batch, transaction processing and time sharing

      * Sever Operating Systems

         * Seve multiple users at once over a network and allow to user to share hardware and software resources
         * Unix, windows200 and Linux

      * Multiprocessor Operating Systems

         * OS designed for Multi-CPU architecture
         * Similar to Sever OS, but have special features for communication and connectivity

      * Personal Computer OS

         * good interface to a single user

      * Real-Time Operating Systems

         * characterized by having time as a key parameter. 
         * Often there are hard deadlines that must be met. 
         * Hard real-time system V.S. Soft real-time system

            * hard is more strict


      * Embedded Operating Systems


         * Smaller system(in microwave ovens)
         * PalmOS and Windows CE


      * Smart Card Operating Systems

         * Smallest Operating system
         * credit card
         * Some ROM holds a JVM, download and use is easy


   * Chapter 2.1 Process

      * Process: an abstraction of a running program 
      * pseudo parallelism: At any instant of time, the CPU running only one program, and it switches from programs very fast giving the illusion that it runs programs parallel

      * True hardware parallelism: Multiprocessor

   * Chapter 2.1.1: The process model


      * In this model, all the runnable software on the computer is organized into a number of sequential processes
      * A process is a executing program, including the current value of the program counter, register, and variables
      * Each process has its own CPU
      * Multiprogramming: The rapid switching back and forth
      * Process is an activity of some kind, the program is the algorithm of that kind. 

         * Program is the recipe, Process is cooking the dish

      * A single processor may be shared among several processes, with some scheduling algorithm being used to determine when to stop work on one process and service a different one


   * 2.1.2:Process creation

      * Four principal events that cause processes to be created

         * System initialization
         * Execution of a process creation system call by a running process
         * A user request to create a new process
         * Initiation of a batch job

      * Foreground processes:

         * Processes that interact with (human) users and perform work for them

      * Background processes, not associated with particular users, but instead have some specific function

         * Daemons

      * Technically, all the process creation is by having an existing process execute a process creation system call. What the process does is execute a system call to create the new process. This system call tells the operating system to create a new process and indicates, directly or indirectly, which program to run it. 
      * In UNIX, there is only one system call to create a new process:fork

         * After the fork,t he two processes,the parent and the child, have the same memory image, the same environment strings and the same open files. 
         * Usually, the child process then executes execve or similar system call to change its memory image and run a new program. 

      * In Windows, the single Win32 function call, CreateProcess, handles both process creation and loading the correct program into the new process.

         * In addition to CreateProdess, Win32 has about 100 other functions for managing and synchronizing process and related topics

      * In both Unix and windows, after a process is created, both the parent and child have their own distinct address spaces. If either process changes a word in its address space, the change is not visible to the other process. 


         * in Unix, the the child's initial address space is a copy of the parents but there are two distinct address spaces involved; no writable memory is shared
         * In windows, the parent's and child's address spaces are different from the start


   * 2.1.3: Process Termination 

      * Normal exit(voluntary)
      * Error exit(voluntary)
      * Fatal error(involuntary)
      * Killed by another process (involuntary)

         * Unix: Killed
         * Windows: TerminateProcess


   * 2.1.4: Process Hierarchies

      * Process only have one parent
      * In UNIX, a process and all of its children and further descendants together form a process group

         * A signal send to the process group-> the individuals may take or not

      * Windows does not have any concept of process hierarchy 

   * 2.1.5: Process states

      * 3 states

         * Running( actually using the CPU at that instant)
         * Ready(runnable; temporarily stopped to let another process run)
         * Blocked(unable to run until some external event happens)

      * Four Transition in between these 3 states

         * process blocks for input
         * scheduler picks another process
         * scheduler picks picks this process
         * input becomes available


   * 2.1.6:Implementation

      * Process table

         * An array of structures, with one entry per process(process control blocks)

            * Contains: Process’ state
            * program counter
            * stack pointer
            * memory allocation
            * status of its open files
            * accounting and scheduling information
            * everything else about the process that must be saved when the process is switched from running to ready to blocked state

         * Process management, memory management, file management 



