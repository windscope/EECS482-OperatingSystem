Operating System-principal and implementation
==
chapter 4
---
- 4.2 Thread abstraction
	- Definition of thread
		- A thread is a __single execution sequence__ that represents __a separately schedulable task__ 
			- Single execution sequence: Each thread executes a sequence of instructions: assignments, conditionals, loops, procedures and so on. 
			- Separately schedulable task: The operating system can run, suspend, or resume a thread at anytime.  
- 4.4 Thread data structure and life circle
	- TCB: thread control Block
	- It holds two 
- 4.6 Implementing 

Chapter 5
---
- 5.4 Condition variables: waiting for a change  
Polling based wait: busy waiting, not good.
	- 5.4.1: condition variable definition
		- CV::wait(lock *lock) :4 atomic step: 1. release the lock 2. suspend the execution of the calling thread, 3. placing the calling thread on the condition variable's waiting list, 4. reacquire the lock before returning from the wait call
		- CV::signal(): Takes one thread off the condition variable's waiting list and marks it as eligible to run. If no threads are on the waiting list, signal has no effect 
		- CV::broadcast: similar to signal, but instead of take one, it take all. Also no effect when no thread is on the waiting list of cv
		- All thread methods can be called while the associated lock is held
			- Why?: A lock is used to wait for a change  shared state, and a lock is designed to protect updates to shared state. 
		- Standard shared object implementation: One mutex lock and 0 or more condition variable
		- Condition variable is memory less: no internal state other than a queue of threads
		- Waiting thread must always wait in loop, since signal and broadcast are not atomic, condition may change after signal. 
	- 5.4.3: Blocking bounded queue
		- Use two conditional variable, example code in page 213
	- 