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
- 4.5 thread life-cycle
	- Init->ready->running->finish. 
	- Running -> waiting -> ready
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
						
	- 5.7 Implementing synchronization objects  
	We have two lock:
		- For uniprocessor: Disabling interrupts: on a single processor, we can make a sequence of instructions atomic by disabling interrupts on that single processor
		- Atomic read-modify-write instructions: On a multiprocessor, disabling interrupts is insufficient to provide atomicity. Instead, architectures provide special instructions to atomically read and update a word of memory. These instructions are globally atomic with respect to the instructions on every processor. 
		- 5.7.1 implementing uniprocessor locks by disabling interrupts
			- This implementation does provide the mutual exclusion property. 
			- busy waiting: No response if the lock is not released
			- Works well in kernel, however would be problematic for user
		- 5.7.2 Implementing uniprocessor Queueing locks
			- If a lock is BUSY when a thread tries to acquire it, the thread moves its TCB onto the lock's waiting list. The thread then suspends itself and switches to the next runnable thread. 
			- Thread systems enforce the invariant that a thread always disables interrupts before performing a context switch. 
			- What happen if we released the lock's spin lock before the call to put to waiting queue. 
				- Another thread may call release the lock and make current thread on ready list which make it both on waiting and on ready list.- Care is needed to prevent a waiting thread from being put back on the ready list until it has completed its context switch
		- 5.7.6 implementing condition variables: Similar to lock. One attention: if lock can pass, we then can implement cv by using a self-contended lock

Chapter 6
---
- 6.4 Multi-object Atomicity  
	- 6.4.1: careful class design
		- decide the atomicity range of the method is crucial. Many functionality need not to be atomic could make the design simple.
	- 6.4.2: Acquire-all/release-all
		- Better interface design has limits. Multiple locks are needed for greater concurrency or just for the functionality. Multi-locked method principles: Acquire-All/Release-All
		- __(1) Acquire-All/Release-All__ : to acquire every lock that may be needed at any point while processing the entire set of operation in the request. Then, once the thread has all of the locks it might need, the thread can execute the request and finally release all locks. 
		- Pros: 
			- Acquire all/release-all allows significant concurrency, When individual requests touch non-overlapping subsets of state protected by different locks, they can proceed in parallel. 
			- Acquire-all/release-all make parallel operation can always express as series operation. __Serializability__.
		- Cons
			- Hard to know what exactly locks will be needed by a request before beginning to process it. 
				- potential Solution: conservatively acquire more locks than needed. 
	- __(2) Two-phase locking__
		- two phase locking refines the acquire-all/release-all pattern to address the  issue of unknown the locks need to have.
		- Locks are adding as needed, and then unlock at the end of the method.  
		- Pros: 
			- Better concurrency. 
			- Serializability
			- Easy to implemented
		- Cons: 
			- Deadlock
			
- 6.5 deadlock
	- A deadlock is a  cycle of waiting among a set of threads, where each thread waits for some other thread in the cycle to take some action. 
	- Deadlock can occur anytime a thread waits for an event that cannot happen because of a cycle of waiting for a resource held by the first thread. 
		- Resource can be: Locks, memory, processing time, disck blocks, or space in a buffer. 
	- Examples
		- Mutually recursive locking
			- A: lock1.lock(); lock2.lock(); B: lock2.lock(); lock1.lock();//this is a n=2 dining philosophy problem
		- Nested waiting
			- Since wait can only release one lock, then when two lock is holding and another thread is waiting two or more lock, then it cannot proceed. 
			- Waiting buffer(special case of nested waiting)
			- Chopstick(Dining philosophy)
	- 6.5.1 Deadlock v.s. Starvation
		- Deadlock and starvation are both liveness concerns. 
			- In starvation, a thread fails take progress for an indefinite period of time. 
			- Deadlock is a form of starvation but with the stronger condition: a group of threads forms a cycle whee none of the threads make progress because each thread is waiting for some other thread in the cycle to take actions. 
		- Differences 
			- Deadlock-> starvation; Starvation \-> deadlock
			- The most important different is that deadlock is a cycle of waiting , whereas starvation is not. 
		- Deadlock and Starvation's subjection issue
			- Might be scheduler, 
	- 6.5.2 Necessary conditions for deadlock  
		4 conditions: 
		- __(1) Bounded(limited) resources__
			- There are finite number of threads that can simultaneously use a resource(others must wait or locked. )
		- __(2) No preemption__ 
			- Once a thread acquires a resource its ownership cannot be revoked until the thread acts to release it. 
		- __(3) Wait while holding__
			- A thread holds one resource while waiting for another. 
			- A.K.A: multiple independent requests 
			- It occurs when a thread first acquires one resource and then tries to acquire another. 
		- __(4) Circular waiting__:
			- There is a set of waiting threads such that each thread is waiting for a resource held by another. 
	Explanations:
		- Eliminate any of these following four condition can make solve the deadlock issue.   
		- However, these 4 deadlock conditions are _necessary_ of deadlock but not _sufficient_ . e.g. new resources may be generated while executing. 
	- 6.5.3 Preventing deadlock
		- 3 ways of overcoming deadlock
			1. Exploit or limit the behavior of the program. 
				- change the behavior of a program to prevent one of the four necessary conditions for deadlock
			2. Predict the future
				- If we can know what threads may or will do, then we can avoid deadlock by having threads wait before they would head into a possible deadlock
			3. Detect and recover
				- Another alternative is to allow threads to recover or 'undo' actions that take a system into a deadlock. 
		- Breaking the 4 necessary conditions
			1. Bounded resources: Provide sufficient resources
				- One way to ensure deadlock freedom is to arrange for sufficient resources to satisfy all threads' demand. 1. e.g.: add a single chopstick to the middle of the table in Dining philosophers. 
			2. No preemption:  Preempt resources
				- Allow the runtime system to forcibly reclaim resources held by a thread. 
				- For example. an operating system can preempt a page of memory from a running process by copying it to disk in order to prevent applications from deadlocking as they acquire memory pages. 
			3. Wait while holding: Release lock when calling out of module
				- For nested modules, each of which has its own lock, waiting on a condition variable in a inner module can lead to a nested waiting deadlock.
					- Solution: restructure a module's code so that no locks are held when calling other modules. 
					- For example, we can change the code on the left to the code on the right, provided that the program does not depend on the three steps occurring atomically. 
							
							left:
							Module::foo(){
								mtx.lock();
								doSomeStuff();
								otherModule->bar();
								mtx.unlock();
							}
							
							Module::doSomeStuff(){
								x = x+1;	
							}
							
							right:
							Module::foo(){		
								doSomeStuff();
								otherModule->bar();
							}
							
							Module::doSomeStuff(){
								mtx.lock();	
								x = x+1;	
								mtx.unlock();
							}
							
				- Circular waiting: lock ordering. 
					- Identify an ordering among locks and only acquire locks in that oder. Especially useful for _mutually recursive waiting_.

	- The Banker's Algorithm for Avoiding Deadlock
		- Acquire-all/ release all. 
			- A general technique to eliminate __wait-while-holding__ condition is to wait until all need resources are available and then to acquire them atomically at the beginning of an operation, rather than incrementally as the operation proceeds. 
			- A thread may not know exactly which resources it will need to complete its work, but it can still acquire all resources that it might need. 
			- Disadvantages: 
				1. The effect on program modularity,
				2.  The challenge of having applications accurately estimate their worst-case needs
				3. The  cost of allocating significantly more resources than may be necessary in the common case. 	
		- Banker's Algorithm
			- __General introduction__: A thread states its maximum resource requirements when it begins a task, but it then acquires and release this resource incrementally as the task runs. The runtime system delays granting some requests to ensure that the system never deadlocks. 
			- __Insight__: A thread may deadlock will not necessarily do so: for some interleaving of requests it will deadlock, but for others it will not.  Delaying the resources requests that leads to deadlock, a system can avoid interleaving that could lead to deadlocks. 
			- __3 state of a deadlock-prone system__:
				- in a __safety state__: For any possible sequence of resource requests, there is at least one _safe sequence_ of processing the requests that eventually success in granting all pending and future requests. 
				- In an __unsafe state__: There is at least one sequence of future resource requests that leads to deadlock no matter what processing order is tried. 
				- In a __deadlocked state__: the system has at least one deadlock. 
			- __Banker's Algorithms's explanation with 3 states__: 
				- Banker's algorithm is to make the system remains in a safe state. 
				- It delays any request that takes the system from a safe to an unsafe state. 
				- 2 reason: 
					1. A system in a safe state controls its own destiny: for any workload, it can avoid deadlock by relying the processing of some requests. 
					2. Once a system enters an unsafe state, it cannot prevent deadlocking. 
			- __Steps to check the safety: __
				1. A state is safe if some sequence of thread executions allows each thread to obtain its maximum resource need, finish its work, and release its resources. 
				2. check if the currently free resources suffice to allow any thread to finish. 
				3. We see if the currently free resources plus any resources held by the thread identified in the first step suffice to allow any other thread to finish
				4. We continue this process until we have identified all threads guaranteed to finish, provided we serve requests in a particular order. 
				5. If 4 is finished, then the thread is true.  
			
		
		
		
			
		
		