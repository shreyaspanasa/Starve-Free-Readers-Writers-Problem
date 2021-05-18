# Starve-Free-Readers-Writers-Problem
Starve-Free Readers Writers Problem

The readers–writers problems are examples of a common computing problem in concurrency.A classical Reader Writer problem is a situation where multiple processes can access and modify a shared file or data structure concurrently. In such a situation in order to avoid the race condition only one writer should be allowed to access the critical section in any point of time and also when no writer is active any number of reader can access the critical section. So to solve this synchronization problem Semaphores are used.
## Starve Free Soltuion
The classical solution of the problem results in starvation of either reader or writer. While proposing the solution only one assumptionis made i.e.  Semaphore preserves the first in first out(FIFO) order when locking and releasing theprocesses( Semaphore uses a FIFO queue to maintain the list of blocked processes).
### Semaphore
Designing a Semaphore with FIRST-IN-FIRST-OUT(FIFO) queue to maintain the list of blocked processes
```cpp
// The code for a FIFO semaphore.
struct Semaphore{
  int value = 1;
  FIFO_Queue* Q = new FIFO_Queue();
}
    
void wait(Semaphore *S,int* process_id){
  S->value--;
  if(S->value < 0){
  S->Q->push(process_id);
  block(); //this function will block the process by using system call and will transfer it to the waiting queue
           //the process will remain in the waiting queue till it is waken up by the wakeup() system calls
           //this is a type of non busy waiting
  }
}
    
void signal(Semaphore *S){
  S->value++;
  if(S->value <= 0){
  int* PID = S->Q->pop();
  wakeup(PID); //this function will wakeup the process with the given pid using system calls
  }
}


//The code for the queue which will allow us to make a FIFO semaphore.
struct FIFO_Queue{
    ProcessBlock* front, rear;
    int* pop(){
        if(front == NULL){
            return -1;            // Error : underflow.
        }
        else{
            int* val = front->value;
            front = front->next;
            if(front == NULL)
            {
                rear = NULL;
            }
            return val;
        }
    }
    void* push(int* val){
        ProcessBlock* blk = new ProcessBlock();
        blk->value = val;
        if(rear == NULL){
            front = rear = n;
            
        }
        else{
            rear->next = blk;
            rear = blk;
        }
    }
    
}

// A Process Block.
struct ProcessBlock{
    ProcessBlock* next;
    int* process_block;
}
```
### Intialization
```cpp
int read_count = 0;                      //Integer representing the number of reader executing critical section
Semaphore turn = new Semaphore();        //seamphore representing the order in which the writer and 
                                         //reader are requesting access to critical section
Semaphore rwt = new Semaphore();         //seamphore required to access the critical section
Semaphore r_mutex = new Semaphore();     //seamphore required to change the read_count variable
```
### Reader's Code
```cpp
do{
/* ENTRY SECTION */
       wait(turn,process_id);              //waiting for its turn to get executed
       wait(r_mutex,process_id);           //requesting access to change read_count
       read_count++;                       //update the number of readers trying to access critical section 
       if(read_count==1)                   // if I am the first reader then request access to critical section
         wait(rwt);                        //requesting  access to the critical section for readers
       signal(turn);                       //releasing turn so that the next reader or writer can take the token
                                           //and can be serviced
       signal(r_mutex);                    //release access to the read_count
/* CRITICAL SECTION */
       
/* EXIT SECTION */
       wait(r_mutex,process_id)            //requesting access to change read_count         
       read_count--;                       //a reader has finished executing critical section so read_count decrease by 1
       if(read_count==0)                   //if all the reader have finished executing their critical section
        signal(rwt);                       //releasing access to critical section for next reader or writer
       signal(r_mutex);                    //release access to the read_count  
       
/* REMAINDER SECTION */
       
}while(true);
```
### Writer's code
```cpp
do{
/* ENTRY SECTION */
      wait(turn,process_id);              //waiting for its turn to get executed
      wait(rwt,process_id);               //requesting  access to the critical section
      signal(turn,process_id);            //releasing turn so that the next reader or writer can take the token
                                          //and can be serviced
/* CRITICAL SECTION */

/* EXIT SECTION */
      signal(rwt)                         //releasing access to critical section for next reader or writer

/* REMAINDER SECTION */

}while(true);
```
## Correctness of Solution
### Mutual Exclusion
This ensures that only a single writer can access the critical section at any moment of time thus ensuring mutual exclusion between the writers and also when the first reader try to access the critical section it has to acquire the rwt mutex lock to access the critical section thus ensuring mutual exclusion between the readers and writers.
### Bounded Waiting
Before accessing the critical section any reader or writer have to first acquire the turn semaphore which uses a FIFO queue for the blocked processes. Thus as the queue uses a FIFO policy, every process has to wait for a finite amount of time before it can access the critical section thus meeting the requirement of bounded waiting.
### Progress Requirement
The code is structured so that there are no chances for deadlock and also the readers and writers takes a finite amount of time to pass through the critical section and also at the end of each reader writer code they release the semaphore for other processes to enter into critical section.
## Explanation on how it works
The  solution works on this method : Any number of readers can simultaneously read the data.  
Once a writer has come, no new process that comes after it can start reading. This is ensured by the writer acquiring the `in_sem` semaphore and not leaving it until the end of its process. This ensures that any new process that comes after this (be it reader or writer) will be queued up on `in_sem`.  Now, from the looks of it, it might seem like this is a method that is biased towards writer and may lead to starvation of readers. However, this is where the FIFO nature of semaphore comes into picture. 
Suppose processes come as `RRRWRWRRR`. Now, by our method the first three readers will first read. Then, when the next process which is a writer takes the `in_sem` semaphore, it won't give it up until it's done. Now, suppose by the time this writer is done writing, the next writer has already arrived. However, it will be queued up on the `in_sem` semaphore. Thus, it won't get to start writing before the process before it, the reader process has completed. Thus, the reader process will get done and then the writer process will again block any more processes from starting and so on.
To explain the writer process further, in case there's no other reader, it doesn't wait at all. Otherwise, it sets `writer_waiting` to true and waits on the `writer_sem` semaphore. Also, to make the `writer_sem` semaphore synchronizable across both the process where one process only executes `wait()` and other only executes `signal()`, we initialize it to `0`.
To explain the reader process further, it firsts increments the `num_started` then reads the data and then increments the `num_completed`. After this, it checks both these variables are equal. If they are and a writer is waiting (found from `writer_waiting`), it signals the `writer_sem` semaphore and finishes.
Thus, it will be pretty clear how we manage to make a starve-free solution for the readers-writers problem with a FIFO semaphore. We can say that all processes will be handled in a FIFO manner. Readers will allow other readers in the queue to start reading with them but writers will block all other processes waiting in the queue from executing before it finishes. In this way, we can implement a reader-writer solution where no process will have to indefinitely wait leading to starvation.
## References
- Abraham Silberschatz, Peter B. Galvin, Greg Gagne - Operating System Concepts
- [Wikipedia](https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)

