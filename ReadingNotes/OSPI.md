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
- 5.5 Designing and implementing shared objects
	- 5.5.1 High level methodology for multithread programming 
		- Steps for multi-threaded programming
			1. Add a lock
			2. Add code to acquire and release the lock
				- Add lock at the very beginning at the public method and release it at very end is not a bad idea. It is inexpensive. 
			3. Identify and add condition variables
				- Ask "when can this method wait"
				- Add each situation in which a method can wait to a condition variable
				- Each "have situation" method should have its own condition variable 
			4. Add loops to wait using the condition variables 
				- Add ```while(...) {cv.wait()}``` loop into each method that identified as potentially needing to wait before returning. 
			5. Add signal and broadcast calls
				- Appropriate for signal when:
					1. At most one waiting rethread can make progress
					2. Any thread waiting on the condition variable can make progress
				- Appropriate for broadcast when:
					1. Multiple waiting threads may all be able to make progress
					2. Different threads are using the same condition variable to wait for different predicted, so some of the waiting threads can make progress but other cannot
	- 5.5.2 Implementation best practices
		1. Consistent structure 
		2. Always synchronize with locks and condition variables. They are best design pattern in multi-threaded programming. Comparing with Semaphores 
		3. Always acquire the lock at the beginning of a method and release it right before the return
		4. Always hold the lock when operating on a condition variable 
		5. Always wait in a while() loop
		6. Never use thread sleep
	- 5.5.3 Three pitfalls
		- Double check locking does not work: always acquire lock at the very beginning is better and simpler design.
		- Others are java, omit here,  
	- 5.6  Three case studies
		- 5.6.1 Readers/writers lock
			- Only one writer, but multiple readers. Readers only read the data structure, writers read and writes the data structure. 
			- Commonly used in databases. 
			- Shared variable  
			
					int activeWriter, waitingWriter;
					int activeReader, waitingReader;
					mutex mtx;
					cv readWait, writeWait;
			- startRead 
					
					void startRead(){
						mtx.lock();
						waitingReader ++;
						while(activeWriter>0 || waitingWriter > 0){
							readWait.wait(mtx);
						}
						waitingReader --;
						activeReader ++;
						mtx.unlock();
					}
			- doneRead 
				
					void doneRead(){
					mtx.lock();
					activeReader --;
					if(activeReader == 0 && waitingWriter > 0){
						writeWait.signal();
					}
					mtx.unlock();
					}
			- startWrite
				
					void startWrite(){
						mtx.lock();
						waitingWriter ++;
						while(activeWriter>0 || activeReader >0){
						writeWait.wait(mtx);
						}
						waitingWriter --;
						activeWriter ++;
						mtx.unlock();
					}
			- doneWrite
				
					void doneWrite(){
						mtx.lock();
						activeWriter --;
						if(waitingWriter>0)
							writeWait.signal();
						else
							readWait.broadcast();
						mtx.unlock();
					}
		- 5.6.2 Synchronization barrier
			- This can be used to implemented mapReduce model
				- Demonstration of using barrier to implement mapReduce
				
						create n threads.
						Create barrier
						
						Each thread executes map
						barrier.checkin()
						
						Each thread send data to reducer
						barrier.checkin();
						
						Each thread execute reducer
						barrier.checkin();
				- Shared data
						
						mutex mtx;
						cv allCheckedin, allLeaving;
						int numThreads, numCheckedin, numLeaving;
				
				- Barrier.checkin();
						
						void checkedin(){
							mtx.lock();
							numCheckedin++;
							if(numChecedin < numThreads){
								while(numCheckedin < numThreads){
									allCheckedin.wait(mtx);
								}
							}
							else{
								numLeaving = 0;
								allCheckedin.broadcast();
							}
							numLeaving ++;
							if(numLeaving < numThreads){
								while(numLeaving <numThreads){
									allLeaing.wait(mtx);
								}
							}
							else{
								allLeaving.broadcast();
								numCheckedin = 0;
							}
							mtx.unlock();
						}