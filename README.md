# Starve Free Readers-Writers Problem



## Table of Contents
1. [Introduction](#one)
2. [Constraints](#two)
3. [Semaphores and Mutex Locks](#three)
4. [First Readers-Writers Problem](#four)
5. [Second Readers-Writers Problem](#five)
6. [Third (Starve-Free) Readers-Writers Problem](#six)

## <a name="one"></a>Introduction
Multiple processes that run in a computer may need to access the same resources at the same time. As such there are two types of processes (users in case of a shared database):
* **Readers** - Never modify database
* **Writers** - Read and modify database

The Readers-Writers Problem arises from the fact that when a writer is modifying the resource, no-one else (reader or writer) can access it at the same time; another writer could corrupt the resource, and another reader could read a partially modified (thus possibly inconsistent) value. Obviously, if two readers access the shared data simultaneously, no adverse affects will result.

This problem has been extensively studied since the 60’s, and is often categorized into three variants:

1. Give readers priority: when there is at least one reader currently accessing the resource, allow new readers to access it as well. This can cause starvation if there are writers waiting to modify the resource and new readers arrive all the time, as the writers will never be granted access as long as there is at least one active reader.
2. Give writers priority: here, readers may starve.
3. Give neither priority: all readers and writers will be granted access to the resource in their order of arrival. If a writer arrives while readers are accessing the resource, it will wait until those readers free the resource, and then modify it. New readers arriving in the meantime will have to wait.

## <a name="two"></a>Constraints 
1. Readers can access database when no writers.
2. Writers can access database when there are no readers and other writers.
3. Only one thread can manipulate state variables at a time.


## <a name="three"></a>Semaphores and Mutex Locks
Semaphores and mutex locks are structures that control access of resources by processes.

### Mutex Lock
**Mutex or Mutual Exclusion Object** is used to give access to a resource to only one process at a time. The mutex object allows all the processes to use the same resource but at a time, only one process is allowed to use the resource. Mutex uses lock-based technique to handle the critical-section problem.

Whenever a process requests for a resource from the system, then the system will create a mutex object with a unique name or ID. So, whenever the process wants to use that resource, then the process occupies a lock on the object. After locking, the process uses the resource and finally releases the mutex object. After that, other processes can create the mutex object in the same manner and use it.

By locking the object, that particular resource is allocated to that particular process and no other process can take that resource. In this way, the process synchronization can be achieved with the help of a mutex object.

### Semaphore
Semaphore is an integer variable S, that is initialized with the number of resources present in the system and is used for process synchronization. It uses two functions to change the value of S i.e. wait() and signal().

Both these functions are used to modify the value of semaphore but the functions allow only one process to change the value at a particular time i.e. no two processes can change the value of semaphore simultaneously. There are two categories of semaphores i.e. Counting semaphores and Binary semaphores.


## <a name="four"></a>First Readers-Writers Problem

The first problem focuses on allowing multiple readers to read the same resource at the same time, while still maintaining mutual exclusion between writers. Since this approach favours readers over writers, this is known as the ***readers-preference*** problem.

In the this problem writers must wait till **all waiting readers** are done reading the resources, before even a single writer can gain access to it. Hence, this method favours readers, and may lead to **starvation of writers**.


## <a name="five"></a>Second Readers-Writers Problem

As mentioned above, the code for the first problem allowed multiple readers, however, allowed writers to starve. The second solution helps prevent starvation of writers and is called the **writer-preference** problem.

## <a name="six"></a> Third Readers-Writers Problem (Starve-Free!)
As both of the above solutions allow either readers or writers to starve, we need a solution that prevents starvation altogether.

The starve-free solution is described with the help of the following code:


#### Global variables
```cpp
// Semaphore to control resource access
ResourceSemaphore = Semaphore()
// Semaphore to preserve ordering of requests
ServiceQueueSemaphore = Semaphore()

ReadMutex = MutexLock() // Mutex lock to control critical section when updating reader_count
int reader_count = 0; // Number of readers currently access the resource
```

#### Implementation for writers
```cpp
writer () {    
    /* BEGIN ENTRY SECTION */

    // Wait for earlier process to finish accessing resource
    ServiceQueueSemaphore.wait();
    // Request exclusive access to resource for writing
    ResourceSemaphore.wait();
    // Once access is granted, allow next item in queue to be serviced
    ServiceQueueSemaphore.signal();

    /* END ENTRY SECTION */

    // Once the access to the resource is granted, we write to it
    /*
        WRITING SECTION
    */

    /* BEGIN EXIT SECTION */
    // Release file from lock after done accessing
    ResourceSemaphore.signal(); 
    /* END EXIT SECTION */
}
```

#### Implementation for readers
```cpp
reader () {
	/* BEGIN ENTRY SECTION */
	// Wait for earlier process to finish accessing resource
	ServiceQueueSemaphore.wait();
		
	// Only one process must be allowed to update the reader count at a time.
	ReadMutex.acquire();
	
	// Add 1 to it because a new process has come to read it.
	reader_count++;
	// Check if this process is the first to read it after a write operation has been done.
    if (reader_count == 1)
	    // If we are the first reader, then lock the resource from all writers (but not other readers).
        ResourceSemaphore.wait();
	
	// Once access is granted, allow next item in queue to be serviced
	ServiceQueueSemaphore.signal();
	
	ReadMutex.release(); // Done updating reader_count
	/* END CRITICAL SECTION */
    
	// Once the access to the resource is granted, we read it
    /*
	    READING SECTION
	*/
	
	/* BEGIN EXIT SECTION */
	// Only one process must be allowded to update the reader count at a time.
    ReadMutex.acquire(); 
    
	/* BEGIN CRITICAL SECTION */
    // This is a critical section because reader_count is a global variable and is being updated.
    
    // Subtract 1 from it because a process has finished reading it.
	reader_count--;       
	
	// Checks if this process is the last to read it.
    if (reader_count == 0)
	    // If we are the last reader, then release the lock from the resource, and
	    // allow writers to write to it if they want.
        ResourceSemaphore.signal();
	/* END CRITICAL SECTION */
    
    ReadMutex.release();           // Release read mutex
   	/* END EXIT SECTION */
}
```

This method prevents starvation of both readers and writers because it follows the following protocol:
- If a reader is reading a resource: 
	- Allow all subsequent readers to read as well.
	- If a writer arrives then it is pushed to the waiting queue. The writer is given access once all readers that arrived before it have finished reading.
- If a writer is writing to a resource:
	- Push any subsequent readers / writers to the waiting queue.

**Note:**
- As a queue follows a FIFO ordering, waiting time for all processes is bound. No process that comes later is given priority over the processes already waiting in queue based on their type of action (read/write).
- A reader_count is maintained for Readers to allow for simultaneous reading of a resource. Such a value is not required for writers because we know that writer_count will always be either 0 or 1.
