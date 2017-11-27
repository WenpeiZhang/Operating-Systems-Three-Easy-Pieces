1. **CPU burst** followed by **I/O burst**
2. **Short-term scheduler** selects from among the processes in ready queue, and allocates the CPU to one of them
* Queue may be ordered in various ways
CPU scheduling decisions may take place when a process:
* Switches from running to waiting state (none preemptive)
* Switches from running to ready state (preemptive)
* Switches from waiting to ready (preemptive)
* Terminates (none preemptive)
3. **Dispatch latency** – time it takes for the dispatcher to stop one process and start another running
4. **Scheduling Criteria:(Optimization: Max, Max, Min, Min, Min respectively)**
* **CPU utilization** – keep the CPU as busy as possible
* **Throughput** – # of processes that complete their execution per time unit
* **Turnaround time** – amount of time to execute a particular process (from the time they come to the time they finish)
* **Waiting time** – amount of time a process has been waiting in the ready queue 
* **Response time** – amount of time it takes from when a request was submitted until the first response is produced, not output (for time-sharing environment)(from the time they come to the first time they start)

*Cautious! Be careful not to mistaken waiting time for response time!!!*

5. **Various CPU-scheduling algorithms (IMPORTANT!!!)(with Ganntt Chart)**

Several concept first:

a. *Ganntt Chart*

b. *Burst time*: the amount of time it takes to finish a certain process

c. *Waiting time* simply equals: 

```
Waiting time = The time a process finished - the time it came - its burst time  
             = the process's turnaround time - its burst time
```

* **First- Come, First-Served (FCFS) Scheduling***
  - *Convoy Effect - short process behind long process* (Consider one CPU-bound and many I/O-bound processes)
* **Shortest-Job-First (SJF) Scheduling**(a special case of the general **priority scheduling algorithm**)
- Associate with each process the length of its next CPU burst
  - Use these lengths to schedule the process with the shortest time
- SJF is optimal – gives minimum average waiting time for a given set of processes
  - The difficulty is knowing the length of the next CPU request (Could ask the user)
- *Preemptive version called shortest-remaining-time-first*
* **Shortest-remaining-time-first** (Preemptive version of SJF)
* **Priority Scheduling**
- *Note that SJF is a special case of the general priority scheduling. It prioritize processes based on their burst time (and SRTF based on their remaining time). No need to worry about remaining time or burst time when you are simply refering to priority scheduling in general. Just look at its priority and decide whether you want it preemptive or not.*
- Problems may occure. A major problem with priority scheduling algorithms is indefinite blocking, or starvation.(processes with low prority can be starved for a long time, think of the rumor of IBM & MIT lab :))
* **Round Robin (RR)** *(Similar to FCFS. Solving the starvation problem)*
- Each process gets a small unit of CPU time (time quantum q), usually 10-100 milliseconds. After this time has elapsed, the process is preempted and added to the end of the ready queue.
- If there are n processes in the ready queue and the time quantum is q, then each process gets 1/n of the CPU time in chunks of at most q time units at once. No process waits more than (n-1)q time units.
- Timer interrupts every quantum to schedule next process
- Performance: q large ⇒ FIFO; q small ⇒ q must be large with respect to context switch, otherwise overhead is too high
- Typically, higher average turnaround than SJF, but better response
* **Multilevel Queue**
- foreground (interactive), background (batch);foreground – RR, background – FCFS
* **Multilevel Feedback Queue**
- Multilevel-feedback-queue scheduler defined by the following parameters:
  -number of queues
  -scheduling algorithms for each queue
  -method used to determine when to upgrade a process
  -method used to determine when to demote a process
  -method used to determine which queue a process will enter when that process needs service
  
6.**Thread Scheduling**
* There are two kinds of threads: *user-level* and *kernel-level threads*
* When threads supported, threads scheduled, not processes
* Many-to-one and many-to-many models, thread library schedules user-level threads to run on LWP
  * Known as *process-contention scope (PCS)* since scheduling competition is within the process
  * Typically don via priority set by programmer
* Kernel thread scheduled onto available CPU is *system-contention scope (SCS)* – competition among all threads in system

7. **Pthread Scheduling**
* API allows specifying either PCS or SCS during thread creation
  * PTHREAD_SCOPE_PROCESS schedules threads using PCS scheduling
  * PTHREAD_SCOPE_SYSTEM schedules threads using SCS scheduling
* Can be limited by OS – Linux and Mac OS X only allow PTHREAD_SCOPE_SYSTEM

8. **Multiple-Processor Scheduling**
* Homogeneous processors within a multiprocessor
* **Asymmetric multiprocessing** – only one processor accesses the system data structures, alleviating the need for data sharing
* **Symmetric multiprocessing (SMP)** – each processor is self- scheduling, all processes in common ready queue, or each has its own private queue of ready processes (Currently, most common)
* **Processor affinity** – process has affinity for processor on which it is currently running
  * Soft affinity
  * Hard affinity
  * Variations including processor sets
*(Note that memory-placement algorithms can also consider affinity)*

9. **Multiple-Processor Scheduling – Load Balancing**
**If SMP**, need to keep all CPUs loaded for efficiency
* **Load balancing** attempts to keep workload evenly distributed
* **Push migration** – periodic task checks load on each processor, and if found pushes task from overloaded CPU to other CPUs
* **Pull migration** – idle processors pulls waiting task from busy processor

10.**Real-Time CPU Scheduling**
* **Soft real-time systems** – no guarantee as to when critical real-time process will be scheduled
* **Hard real-time systems** – task must be serviced by its **deadline**
* Two types of **latencies** affect performance
  - **Interrupt latency** - time from *arrival of interrupt* to *start of routine that services interrupt*
  - **Dispatch latency** - time for schedule to take current process off CPU and switch to another
* Conflict phase of dispatch latency:
  - Preemption of any process running in kernel mode
  - Release by low- priority process of resources needed by high-priority processes（#insert pic ppt 38）
  
11.**Priority-based Scheduling** (Point 10 Cont.)
* For real-time scheduling, scheduler must support preemptive, priority-based scheduling (But only guarantee soft real-time, which means it cannot guarantee all tasks to be finished by their deadlines)
* For hard real-time must also provide ability to meet **deadlines**
* Processes have new characteristics: periodic ones require CPU at constant intervals *(They require CPU once in a while, and the periodity is fixed, thus implying the deadline of the first cycle of process A is also the starting point of process A's second cycle )*
  - Has **processing time t (Also said to be the "Worst-case Execute Time", known as "WCET"), deadline d, period p**
  - *0 ≤ t ≤ d ≤ p* (deadline is more like a time point while WCET/processing time is like a time slot)
  - **Rate** of periodic task is *1/p*
  
12. **Rate Montonic Scheduling (IMPORTANT!!!)**
* A priority is assigned based on the inverse of its period (Shorter periods = Higher Priority, Longer periods = Lower Priority)
* It is a static priority algorithm **(Priority is fixed!)**
* **Preemptive**
* Can cause missed deadline, since it cannot guarantee hard real-time systems
  
13. **Earliest Deadline First Scheduling (EDF) (IMPORTANT!!!)**
* Priorities are assigned according to deadlines (the earlier the deadline, the higher the priority;the later the deadline, the lower the priority)
* **Priority is not fixed!** It can change over time, base on different deadlines!
* **Preemptive**

