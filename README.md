# Starve Free Readers-Writers Problem


## What is the Readers-Writers Problem?
Multiple processes that run in a computer may need to access the same resources at the same time. Let us take a file for example. It is possible that at the same time, some processes would **write** to that file and others would **read** from that file. Since during reading, the file is not updated, multiple processes reading a file at the same time will not result in any error. However during writing, the contents of a file are modified, if more than one process writes a file at the same time, then there is an error. 

## What is Starve Free?
Let us first understand what **starvation** is.
In a computer there may be many processes being executed at the same time, it may be possible that the computer is **not** able to execute all the process **instantly** (i.e. as soon as the CPU receives the process request) because of CPU limitations and other factors. In such conditions, the **CPU scheduler** picks the processes for execution and the picked processes are executed. 
Now, it may be possible that a particular process is never picked, thus the process keeps waiting to be executed. This is called **starvation**. An algorithm that avoids starvation is called **Starve Free**.






## Starve Free Readers-Writers Algorithm

The pseducode for the starve free readers-writer problem can be found in the ``readers.txt`` and ``writers.txt`` files. The implementation of read request is shown in ``readers.txt`` and write requests is shown in ``writers.txt``. 
Here is a detailed explanation of the code.

#### Global Variables used
|Name|Initial Value|Description|
|----|-------------|-----------|
|counter_in|0|It keeps a track of the number of read processes that have entered the critical section and are executing read operation and also helps in mutual exclusion|
|counter_out|0|It keeps a track of the number of read processes that have finished executing the read operation and also helps in mutual exclusion|
|wait|0|If wait is 1 it means that process is waiting for write operation|

#### Semaphores used
|Name|Initial Value|Description|
|----|-------------|-----------|
|in|1|It is used as a mutex lock to prevent mutual access to counter_in variable|
|out|1|It is used as a mutex lock to prevent mutual access to counter_out variable|
|wrt|0|It used to prevent write process from executing when one/more readers are reading the file|

#### Readers Section
```Pseudocode
Wait in
counter_in++
Signal in
.
.
.
[Execute critical section here]
.
.
.
Wait out
counter_out++
if (wait==1 && counter_in==counter_out)
    Signal wrt
Signal out 
```

First we use the in semaphore for updating counter_in with mutual exclusion, therefore only one process can update this at a time.
Now we execute the critical section, which here is the reading of the file.
Now we use the out semaphore, again for mutual exclusion and increase counter_out, next we check if any process is waiting to write into the file. If yes, then we signal the wrt semaphore.


#### Writers Section
```Pseudocode
Wait in
Wait out
if (counter_in==counter_out)
    Signal out
else
    wait=1
    Signal out
    Wait wrt
    wait=0
.
.
.
[Execute critical section here]
.
.
.
Signal in 
```

First we wait on the in and out semaphores. 
Next we check if counter_in is equal to counter_out, 
    * If this is true, then it means that no process is reading the file, in this case we signal the out semaphore, then perform te criticl section, which is writing to the file and then signal the in semaphore.
    * However, if the condition is not true, it means that there are certain processes that are reading the file, so we set wait as 1, which indicates that this process is waiting to write onto the file, then signal the out semaphore. After this we wait on the wrt semaphore, the wrt semaphore is initially 0 and is signalled by the readers section under the condition that wait is 1 and counter_in is equal to counter_out. As soon as wrt is signalled, we make wait 0, then perform the critical section, which is writing to the file and then signal the in semaphore.

#### How does the algorithm work?

Consider a case when write statement is first, now we wait on in and out, check if counter_in is equal to counter_out or not, since initially both are set to 0, this condition is true, we release out then write to the file and then release in.
Now let us consider that another write/read request has been made when this process was writing to the file. Since the in semaphore is not yet released and both the read and write request first wait for the in semaphore, therefore none of them can enter the critical section and hence mutual exclusionis prevented.

Let us consider a case when some read requests have been made and a write request arrives. Again the write request checks if counter_in is equal to counter_out. If it is equal then no process is reading and it executes as stated above. However, if they are not equal, it sets wait to 1 and waits on wrt. Also if we have any more read requests, the read requests do not execute because they keep waiting on in and in will be released only by the write process. As soon as all processes finish reading, wrt is signalled, then write process writes onto the file and finally signals in . Now any other process (read or write) that was waiting on the in semaphore can resume it's execution.