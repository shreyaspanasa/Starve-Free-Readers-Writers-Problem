# Starve-Free-Readers-Writers-Problem
Starve-Free Readers Writers Problem

The readers–writers problems are examples of a common computing problem in concurrency.A classical Reader Writer problem is a situation where multiple processes can access and modify a shared file or data structure concurrently. In such a situation in order to avoid the race condition only one writer should be allowed to access the critical section in any point of time and also when no writer is active any number of reader can access the critical section. So to solve this synchronization problem Semaphores are used.
## Starve Free Soltuion
The classical solution of the problem results in starvation of either reader or writer. While proposing the solution only one assumptionis made i.e.  Semaphore preserves the first in first out(FIFO) order when locking and releasing theprocesses( Semaphore uses a FIFO queue to maintain the list of blocked processes).
