
# 1: The difference between process and thread:
```In simple terms, the process is likened to a train, and the thread is a train car.
Threads travel under processes.
A process consists of many threads. (A train has multiple carriages).
````

#### It is difficult to share data between different processes, but the communication between threads is much simpler (station transfer, car change).
#### Processes consume more computer resources than threads.
#### Different processes do not affect each other, but if a thread hangs up, it will cause a process to hang up (the carriage is broken).

# 2: Inter-process communication method:
* Pipe: such as `cat catalina.out | grep -C "10" "id"` under linux.
* Signal: It's more complicated, I don't want to write it, I won't ask in detail.
* Message queue: The message queue is a linked list of messages. It overcomes the shortcomings of the limited semaphore in the above two communication methods. Processes with write permissions can add new information to the message queue according to certain rules; read the message queue. Processes with privileges can read information from the message queue.
* Shared memory: Arguably the most useful way to communicate between processes. It enables multiple processes to access the same memory space, and different processes can see the update of the data in the shared memory in the other process in time. This method needs to rely on some kind of synchronization operation, such as mutex and semaphore;
* Semaphore: the most commonly used and most useful, it enables multiple processes to access the same memory space, and different processes can see the update of the data in the shared memory in the other process in time. Such as: `kill -9 #{id}, control+C` under linux.
 * Socket (socket): This is a more general inter-process communication mechanism, which can be used for inter-process communication between different machines in the network, and is widely used. `For example, 3306 of mysql, request to 3306 through the network`.
# 3: Thread synchronization method:
* Mutual exclusion (lock, synchronized): Using the mutex object mechanism, only the thread that owns the mutex object has the right to access public resources. Because there is only one mutex object, it is guaranteed that common resources will not be accessed by multiple threads at the same time.
Semaphore: It allows multiple threads to access the same resource at the same time, but needs to control the maximum number of threads that access this resource at the same time.

* Event (notify, wait): Keep multi-thread synchronization by means of notification operation, and can also easily implement multi-thread priority comparison operation.
# 4: Process scheduling algorithm:
 * First-come, first-served: `non-preemptive scheduling algorithm`, scheduling according to the order of requests. `It is good for long jobs, but not good for short jobs`, which may cause the waiting time of short jobs to be too long. It is also bad for `I/O-intensive processes, which have to be requeued after each I/O operation`.
 * Short job priority: `non-preemptive scheduling algorithm`, the job is very short first.
 * Shortest remaining time first: The `preemptive` version of the shortest job first, scheduled in the order of remaining running time. When a new job arrives, its entire running time is compared to the remaining time of the current process. If the new process needs less time, suspend the current process and run the new process. Otherwise the new process waits.
 * Time slice rotation: `Arrange all ready processes into a queue according to the principle of FCFS`, and allocate CPU time to the queue head process each time it is scheduled, which can execute a time slice. When the time slice is used up, a clock interrupt is issued by the timer, and the scheduler stops the execution of the process and sends it to the end of the ready queue, while continuing to allocate CPU time to the process at the head of the queue.
* This leads to another problem: efficiency has a lot to do with the size of the time slice.
#5: Deadlock:
* cause:
    * Mutual exclusion: At least one resource is non-shared, that is, only one process can access it at a time. For example, a goddess is chased by many people, but can only be with one at the same time. After breaking up, you can consider sharing with others The suitors are together, here the goddess is replaced by resources, and the suitors are the process.
    * Occupy and wait: A process must occupy at least one resource and wait for another resource, which is occupied by other processes;
    * Non-preemptive: The process cannot be preempted and must be automatically released after the process completes its task.
    * Circular waiting: Multiple processes form a ring waiting for each other.
* Prevent deadlock:
    * Deadlock prevention: just break one of the four causes of deadlock
    * Break mutual exclusion conditions: allow processes to access certain resources at the same time. The goddess can talk about more.
    * Break the hold and wait condition: The resource pre-allocation strategy can be implemented.
    * Break the non-preemption condition: allow the process to forcefully grab certain resources from the occupant.
    * Break the circular waiting condition: implement an orderly allocation of resources.
* Deadlock avoidance:
    * The basic idea of ​​deadlock avoidance is to dynamically detect the resource allocation state to ensure that the circular waiting condition does not hold, thereby ensuring that the system is in a safe state. The so-called safe state means: if the system can allocate resources to each process in a certain order (not exceeding its maximum value), then the system state is safe, in other words, if there is a safe sequence, then the system is in a safe state . Resource Allocation Graph Algorithm and Banker's Algorithm are two classic deadlock avoidance algorithms that can ensure that the system is always in a safe state. Among them, the application scenario of the resource allocation graph algorithm is that there is only one instance of each resource type (application side, allocation side, demand side, and allocation is allowed without forming a ring), while the banker's algorithm is applied to each resource type that can have multiple instances. Scenes.
* Deadlock release:
++ Two commonly used methods for deadlock release are process termination and resource preemption. At this time, we need to consider three issues: ++
    * Choose a victim
    * rollback: roll back to a safe state
    * hungry
# 6: Status of the process:
* Ready state: The process has obtained the required resources other than the processor and is waiting to allocate processor resources;
* Running state: Occupy processor resources to run, the number of processes in this state is less than or equal to the number of CPUs;
* Blocking state: The process waits for a certain condition and cannot execute until the condition is met;
# 7: The state of the thread:
*new
* ready
* running
* blocked
* exit
Specific reference notes

#8: Fair and Unfair Locks
* Fair lock: Multiple threads acquire locks in the order in which they apply for locks, and the threads will directly enter the queue to queue, and the first place in the queue can always get the lock.
    * Advantage: All threads can get resources and will not starve to death in the queue.
    * Disadvantage: The throughput will drop a lot. Except for the first thread in the queue, other threads will be blocked, and the overhead of cpu awakening the blocked thread will be very large.
* Unfair lock: When multiple threads acquire a lock, they will try to acquire it directly. If they cannot acquire it, they will enter the waiting queue. If they can acquire it, they will acquire the lock directly.
    * Advantages: It can reduce the overhead of CPU wake-up threads, the overall throughput efficiency will be higher, and the CPU does not have to wake up all threads, which will reduce the number of wake-up threads.
    * Disadvantage: It may cause the thread in the middle of the queue to be unable to obtain the lock or to obtain the lock for a long time, resulting in starvation.
# 9: Optimistic locks pessimistic locks (note the distinction, use go to introduce)
* The basic concepts are as follows:

    * Optimistic locking assumes that data will not cause conflicts in general, so when the data is submitted for update, the data conflict will be formally detected. If a conflict is found, the wrong information will be returned to the user, allowing the user to decide How to do it.
    * When modifying a piece of data in a database, in order to avoid being modified by others at the same time, the best way is to directly lock the data to prevent concurrency. This method of locking before modifying data with the help of the database lock mechanism, and then modifying it is called pessimistic concurrency control (synchronized in java).
 * Implementation principle
    * Pessimistic lock: mainly through shared locks or exclusive locks, first lock the data, and operate (in fact, lock and unlock). The approximate process is.
        * <font color="#dd0000">*1:ab two threads, controlling two transactions, transaction a locks the shared data successfully, and b is blocked*
        * 2: Transaction a commits, the lock is released, transaction b adds a lock to the data, and then starts executing *:</font><br />
    * Optimistic locking: CAS (compare and exchange)
        * <font color="#dd0000"> A CAS operation consists of three operands - the memory location to be read and written (V), the expected old value to be compared (A), and the new value to be written (B). If the value of memory location V matches the expected original value of A, then the processor will automatically update the location value to the new value of B. Otherwise the processor does nothing. In either case, it returns the value at that location before the CAS instruction. </font><br />
    * Code:
````go
import (
"fmt"
"sync"
"sync/atomic"
"time"
)

func IncreValue1() { //Do not lock
value1++
}

func IncreValue2() { //Optimistic lock
atomic.AddInt32(&value2, 1)
}

func IncreValue3() { //pessimistic lock
lock.Lock()
value3++
lock.Unlock()
}

var value1, value2, value3 int32
var lock sync.Mutex

func main() {
//Open 1000 threads and perform auto-increment operations
for i := 0; i < 1000; i++ {
go IncreValue1()
go IncreValue2()
go IncreValue3()
}
// print the result
time.Sleep(1000)
fmt.Println("Thread unsafe: ", value1) //Thread unsafe: 988
fmt.Println("Optimistic lock:", value2) //Optimistic lock: 1000
fmt.Println("pessimistic lock: ", value2) //pessimistic lock: 1000
}
````
# 10: Memory Management
* Why have virtual memory
> The physical main memory space is limited, so generally modern operating systems will find a way to put some memory blocks on the disk, and then load them into the main memory when they are used, but for user programs, it is not necessary to pay attention to the actual physical memory .
<font color="#dd0000">Simply put, virtual memory is a mechanism provided by the operating system to map the virtual addresses of different processes with the physical addresses of different memories. </font><br />

* Memory segmentation
> A program is composed of several logical segments, such as code segment, data segment, stack segment, and heap segment. Different segments have different attributes, so these segments are separated in the form of Segmentation.
<font color="#dd0000">The virtual address under the segmentation mechanism consists of two parts, the segment number and the offset within the segment.
Virtual addresses and physical addresses are mapped through a segment table, which mainly includes segment numbers and segment boundaries. </font><br />
![Insert image description here](https://img-blog.csdnimg.cn/e2ed93e467ee477f86660ce4af086379.png)
* Memory paging
> Fragmentation is to cut the entire virtual and physical memory space into sections of fixed size. Such a continuous and fixed-size memory space is called a page.
![Insert image description here](https://img-blog.csdnimg.cn/0108486cf2104fe3893a3f2e3958cc18.png)
