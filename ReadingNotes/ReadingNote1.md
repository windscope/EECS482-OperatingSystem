


   * Chapter 1.1
   * 
      * two main function of OS: extending the machine & managing resources
      * extending machine
      * 
         * Hardware control are complex, so OS present the user with the equivalent of an extended machine or virtual machine that is easier to program than the underlying hardware
      * resource manager
      * 
         * Time multiplexing(share)
         * 
            * Determining how the resource is time multiplexed-who goes next and for how long—is the task of the operating system
         * Space multiplexing(share)
         * 
            * Allocating disk space and keeping track of who is using which disk hocks is a typical operating system resource management task
   * Chapter 1.3 The operating system zoo
   * 
      * Mainframe operating systems
      * 
         * Distinguishing themselves from personal OS in term of their I/O capacity
         * Batch, transaction processing and time sharing
      * Sever Operating Systems
      * 
         * Seve multiple users at once over a network and allow to user to share hardware and software resources
         * Unix, windows200 and Linux
      * Multiprocessor Operating Systems
      * 
         * OS designed for Multi-CPU architecture
         * Similar to Sever OS, but have special features for communication and connectivity
      * Personal Computer OS
      * 
         * good interface to a single user
      * Real-Time Operating Systems
      * 
         * characterized by having time as a key parameter. 
         * Often there are hard deadlines that must be met. 
         * Hard real-time system V.S. Soft real-time system
         * 
            * hard is more strict
      * Embedded Operating Systems

      * 
         * Smaller system(in microwave ovens)
         * PalmOS and Windows CE

      * Smart Card Operating Systems
      * 
         * Smallest Operating system
         * credit card
         * Some ROM holds a JVM, download and use is easy
   * Chapter 2.1 Process
   * 
      * Process: an abstraction of a running program 
      * pseudo parallelism: At any instant of time, the CPU running only one program, and it switches from programs very fast giving the illusion that it runs programs parallel

      * True hardware parallelism: Multiprocessor
   * Chapter 2.1.1: The process model

   * 
      * In this model, all the runnable software on the computer is organized into a number of sequential processes
      * A process is a executing program, including the current value of the program counter, register, and variables
      * Each process has its own CPU
      * Multiprogramming: The rapid switching back and forth
      * Process is an activity of some kind, the program is the algorithm of that kind. 
      * 
         * Program is the recipe, Process is cooking the dish
      * A single processor may be shared among several processes, with some scheduling algorithm being used to determine when to stop work on one process and service a different one

   * 2.1.2:Process creation
   * 
      * Four principal events that cause processes to be created
      * 
         * System initialization
         * Execution of a process creation system call by a running process
         * A user request to create a new process
         * Initiation of a batch job
      * Foreground processes:
      * 
         * Processes that interact with (human) users and perform work for them
      * Background processes, not associated with particular users, but instead have some specific function
      * 
         * Daemons
      * Technically, all the process creation is by having an existing process execute a process creation system call. What the process does is execute a system call to create the new process. This system call tells the operating system to create a new process and indicates, directly or indirectly, which program to run it. 
      * In UNIX, there is only one system call to create a new process:fork
      * 
         * After the fork,t he two processes,the parent and the child, have the same memory image, the same environment strings and the same open files. 
         * Usually, the child process then executes execve or similar system call to change its memory image and run a new program. 
      * In Windows, the single Win32 function call, CreateProcess, handles both process creation and loading the correct program into the new process.
      * 
         * In addition to CreateProcess, Win32 has about 100 other functions for managing and synchronizing process and related topics
      * In both Unix and windows, after a process is created, both the parent and child have their own distinct address spaces. If either process changes a word in its address space, the change is not visible to the other process. 

      * 
         * in Unix, the child's initial address space is a copy of the parents but there are two distinct address spaces involved; no writable memory is shared
         * In windows, the parent's and child's address spaces are different from the start
   * 2.1.3: Process Termination 
   * 
      * Normal exit(voluntary)
      * Error exit(voluntary)
      * Fatal error(involuntary)
      * Killed by another process (involuntary)
      * 
         * Unix: Killed
         * Windows: TerminateProcess
   * 2.1.4: Process Hierarchies
   * 
      * Process only have one parent
      * In UNIX, a process and all of its children and further descendants together form a process group
      * 
         * A signal send to the process group-> the individuals may take or not
      * Windows does not have any concept of process hierarchy 
   * 2.1.5: Process states
   * 
      * 3 states
      * 
         * Running( actually using the CPU at that instant)
         * Ready(runnable; temporarily stopped to let another process run)
         * Blocked(unable to run until some external event happens)
      * Four Transition in between these 3 states
      * 
         * process blocks for input
         * scheduler picks another process
         * scheduler picks picks this process
         * input becomes available
   * 2.1.6:Implementation
   * 
      * Process table
      * 
         * An array of structures, with one entry per process(process control blocks)
         * 
            * Contains: Process’ state
            * program counter
            * stack pointer
            * memory allocation
            * status of its open files
            * accounting and scheduling information
            * everything else about the process that must be saved when the process is switched from running to ready to blocked state
         * Process management, memory management, file management 
   * 4.2.0 swapping
   * 
      * With batch system, organizing memory into fixed partitions is simple and effective. 
      * Two general memory management method 
      * 
         * Swapping, consist of bringing in each process in its entirety, running it for a while, then putting it back on the disk
         * Virtual memory, allows programs to run even when they are only partially in main memory
      * The swapping process is well defined in figure 4-5 of the OS book 
      * memory compaction
      * 
         * when swapping creates multiple holes in memory, it is possible to combine them all into one big one by moving all the process downward as far as possible. 
      * Dynamic memory allocation of processes can be a issue when compacting memory
      * 
         * solution: allocate a little extra memory whenever a process is swapped in or moved 
         * If memory run out, process need to move td a hole with enough space, swapped out of memory until a large enough hole can be created or killed
   * 4.2.1 Memory management with Bitmaps

   * 
      * two way to keep track of memory usage: bitmaps and free lists
      * bitmap is dividing memory up into allocation units, the size of the unit can vary. Corresponding to each allocation unit is a bit in the bitmap, which is 0 if the unit is free and 1 if  it is occupied
      * Issue
      * 
         * Search in bitmap is slow since we need to find a k-memory-allocated-process a k consecutive 0s, and this is particularly slow because straddle word boundaries
   * 4.2.2 Memory management with Linkedlist
   * 
      * Because of the issue of slowness of bitmap, a linked list is a much more practical approach
      * Generally the naive approach is sorted the list by memory, with 
      * 
         * 1. Whether is a process or a hole
         * 2.  Memory occupied-start-location
         * 3. Memory occupation size
      * A double linked list is preferred than single one because it is easier to find the holes when terminated
      * 4 possible situation and memory manipulation when a process is terminated:
      * 
         * PA+X+PB -> PA+H+B
         * PA+X+H -> PA+H
         * H+X+B -> H+B
         * H+X+H -> H
      * 3 algorithm with naive(memory location sorted) memory linked list
      * 
         * First fit
         * 
            * first list is put the process into the first large enough hole
         * Next fit
         * 
            * Next fit is to put the process into the first large enough hole, but next time not start from the beginning but from the last stopped position
            * Worse than First fit
         * Best fit
         * 
            * Go through the entire list find the best fit hole(smallest yet large enough)
            * Worse than first fit, many fragile holes
      * Other Algorithm with complex memory linked list
      * 
         * Complex here means not sort the memory with location and keep multiple lists
         * Worst fit algorithm
         * 
            * Have 2 list: holes, and process, sorted by the memory size, 
            * Always fit the largest hole
            * Very complex when free the memories
         * Quick fit
         * 
            * Have multiple hole lists with common sizes request: 4KB, 8KB, …
            * same issue as worst fit, complex when free the list and put back to holes list
