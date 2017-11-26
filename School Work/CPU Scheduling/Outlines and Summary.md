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
```Waiting time = The time a process finished - the time it came - its burst time = the process's turnaround time - its burst time```

* **First- Come, First-Served (FCFS) Scheduling**
*
