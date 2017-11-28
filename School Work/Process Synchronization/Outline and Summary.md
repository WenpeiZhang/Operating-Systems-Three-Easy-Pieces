## Concurrency: An Introduction
1. Each threads has its own private set of registers it uses for computation; thus, if there are two threads that are running on a single processor, when switching from running one (T1) to running the other (T2), a **context switch** must take place.

2. Difference between threads and processes:
* in the context switch we perform between threads as compared to processes: the address space remains the same (i.e., there is no need to switch which page table we are using). 
  * With processes, we saved state to a **process control block (PCB)**; now, we’ll need one or more **thread control blocks (TCBs)** to store the state of each thread of a process.
  
* One other major difference between threads and processes concerns the **stack**. However, in a multi-threaded process, instead of a single stack in the address space, there will be one *per thread*.
  * any stack-allocated variables, parameters, return values, and other things that we put on the stack will be placed in what is sometimes called **thread-local storage**, i.e., the stack of the relevant thread.

3. The reason for using threads:
* **Parallelism**: The task of transforming your standard **single-threaded** program into a pro- gram that does this sort of work on multiple CPUs is called **parallelization**.
* To avoid blocking program progress due to slow I/O. Threading eanables **overlap** of I/O with other activities *within* a single program.
* Threads share an address space and thus make it easy to share data

4. Thread Creation
* Once a thread is created, **it may start running right away** (depending on the whims of the scheduler); **alternately, it may be put in a “ready” but not “running” state and thus not run yet.** (Of course, on a multiprocessor, the threads could even be running at the same time, but let’s not worry about this possibility quite yet.)What runs next is determined by the OS scheduler.
  * We also could even see “B” printed before “A”, if, say, the scheduler decided to run Thread 2 first even though Thread 1 was created earlier; *there is no reason to assume that a thread that is created first will run first.*
  
* `pthread_create(...)`creates threads, After creating the two threads (let’s call them T1 and T2), the main thread calls `pthread join(...)`, which *waits for a particular thread to complete*. It does so twice, thus ensuring T1 and T2 will run and com- plete before finally allowing the main thread to run again

5. How threads interact when they access shared data

a. example: 2million --> 1900000 

Thread 1 load the value of *counter* into its register *eax*.Thus, *eax=50* for Thread 1. Then it adds one to the register; thus *eax=51*. **Now, something unfortunate happens**: a timer interrupt goes off; thus, the OS saves the state of the currently running thread (its PC, its registers including eax, etc.) to the thread’s *TCB*.......

b. The example above demonstrate a **race condition: the results depend on the timing execution of the code.** With some bad luck (i.e., context switches that occur at untimely points in the execution), we get the wrong result. Instead of a nice deterministic computation, we call this result indeterminate, where it is not known what the output will be and it is indeed likely to be different across runs.

c. Multiple threads executing this code can result in a **race condition**, we call this code a **critical section**. A **critical section** is **a piece of code (code segment)** that accesses a *shared variable (or more generally, a shared resource)* and **must not** be concurrently executed by more than one thread.

d. What we really want for this code is what we call **mutual exclusion**. This property guarantees that if one thread is executing within the critical section, the others will be prevented from doing so.

6. The wish for **atomocity**
* **Atomically**, in this context, means “as a unit”, which sometimes we take as “all or none.” What we’d like is to execute the three instruction sequence atomically:there is no in-between state.
* what we will instead do is ask the hardware for a few useful instructions upon which we can build a general set of what we call **syn- chronization primitives.**

### Learning to build multi-threaded code that accesses critical sections in a synchronized and controlled manner is the main theme of this chapter.

#### Two main problems to solve:
a. Interactions of threads when accessing shared variables and the need to support atomicity for critical sections. (Mutex/Lock)
b. Interactions of threads when one thread must wait for another to complete some action before it continues. (Condition variables)

#### Key Concurrency Terms
1. A **critical section** is a piece of code that accesses a shared resource, usually a variable or data structure.

2. A **race condition** arises if multiple threads of execution enter the critical section at roughly the same time; both attempt to update the shared data structure, leading to a surprising (and perhaps un- desirable) outcome.

3. An **indeterminate** program consists of one or more race conditions; the output of the program varies from run to run, depending on which threads ran when. The outcome is thus not **deterministic**, something we usually expect from computer systems.

4.To avoid these problems, threads should use some kind of **mutual exclusion** primitives; doing so guarantees that only a single thread ever enters a critical section, thus avoiding races, and resulting in deterministic program outputs.

## Deadlock
1. **Conditions for Deadlock**

Four conditions need to hold for a deadlock to occur:
* **Mutual exclusion**: Threads claim exclusive control of resources that they require (e.g., a thread grabs a lock).
* **Hold-and-wait**:Threads hold resources allocated to them (e.g.,locks that they have already acquired) while waiting for additional resources (e.g., locks that they wish to acquire).
* **No preemption**: Resources (e.g., locks) cannot be forcibly removed from threads that are holding them.
* **Circular wait**: There exists a circular chain of threads such that each thread holds one or more resources (e.g., locks) that are being requested by the next thread in the chain.

*If any of these four conditions are not met, deadlock cannot occur. Thus, we first explore techniques to prevent deadlock; each of these strate- gies seeks to prevent one of the above conditions from arising and thus is one approach to handling the deadlock problem.


### Prevention
 #### Circular Wait
 Probably the most practical prevention technique (and certainly one that is frequently employed) is to write your locking code such that you never induce a circular wait. 
 
 The most straightforward way to do that is to pro- vide a total ordering on lock acquisition. For example, if there are only two locks in the system (L1 and L2), you can prevent deadlock by always acquiring L1 before L2. Such strict ordering ensures that no cyclical wait arises; hence, no deadlock.

Of course, in more complex systems, more than two locks will ex- ist, and thus total lock ordering may be difficult to achieve (and per- haps is unnecessary anyhow). Thus, a partial ordering can be a useful way to structure lock acquisition so as to avoid deadlock.

*ENFORCE LOCK ORDERING BY LOCK ADDRESS

#### Hold-and-wait

The hold-and-wait requirement for deadlock can be avoided by acquiring all locks at once, atomically.

It requires that any time any thread grabs a lock, it first acquires the **global** prevention lock. For example, if another thread was trying to grab locks L1 and L2 in a dif- ferent order, it would be OK, because it would be holding the prevention lock while doing so, thus guarantees that no untimely thread switch can occur in the midst of lock acquisition and thus deadlock can once again be avoided.

Note that the solution is **problematic** for a number of reasons. As before, encapsulation works against us: when calling a routine, this ap- proach requires us to know exactly which locks must be held and to ac- quire them ahead of time. This technique also is likely to decrease con- currency as all locks must be acquired early on (at once) instead of when they are truly needed.

1. Three requirements for the **critical-section** problem are: **Mutual exclusion, Progress, Bounded waiting.**(Exiercise 6.1，6.2)

**Mutual Exclusion** - If process Pi is executing in its critical section, then no other processes can be executing in their critical sections

**Progress** - If no process is executing in its critical section and there exist some processes that wish to enter their citical section, then the selection of the processes that will enter the critical section next cannot be postponed indefinitely

**Bounded Waiting** - A bound most exist on the number of times that other processes are allowed to enter their critical section and before that request is granted
