1. Three requirements for the **critical-section** problem are: **Mutual exclusion, Progress, Bounded waiting.**(Exiercise 6.1，6.2)

**Mutual Exclusion** - If process Pi is executing in its critical section, then no other processes can be executing in their critical sections

**Progress** - If no process is executing in its critical section and there exist some processes that wish to enter their citical section, then the selection of the processes that will enter the critical section next cannot be postponed indefinitely

**Bounded Waiting** - A bound most exist on the number of times that other processes are allowed to enter their critical section and before that request is granted
