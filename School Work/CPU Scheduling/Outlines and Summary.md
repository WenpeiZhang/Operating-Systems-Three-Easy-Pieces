1. **CPU burst** followed by **I/O burst**
2. **Short-term scheduler** selects from among the processes in ready queue, and allocates the CPU to one of them
* Queue may be ordered in various ways
CPU scheduling decisions may take place when a process:
* Switches from running to waiting state (none preemptive)
* Switches from running to ready state (preemptive)
* Switches from waiting to ready (preemptive)
* Terminates (none preemptive)
3. **Dispatch latency** – time it takes for the dispatcher to stop one process and start another running
4. Scheduling Criteria:(Optimization: Max, Max, Min, Min, Min respectively)
* **CPU utilization** – keep the CPU as busy as possible
* **Throughput** – # of processes that complete their execution per time unit
* **Turnaround time** – amount of time to execute a particular process (from the time they come to the time they finish)
* **Waiting time** – amount of time a process has been waiting in the ready queue 
* **Response time** – amount of time it takes from when a request was submitted until the first response is produced, not output (for time-sharing environment)(from the time they come to the first time they start)

*Cautious! Be careful not to mistaken waiting time for response time!!!*

5. Various CPU-scheduling algorithms (IMPORTANT!!!)(with Ganntt Chart)

Several concept first:

a. *Ganntt Chart*

b. *Burst time*: the amount of time it takes to finish a certain process

c. *Waiting time* simply equals: 

```
Waiting time = The time a process finished - the time it came - its burst time  
             = the process's turnaround time - its burst time
```

* **First- Come, First-Served (FCFS) Scheduling**
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
* **Round Robin (RR)** *(Solving the starvation problem)*
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

