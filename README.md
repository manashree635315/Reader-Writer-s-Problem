# Reader Writer's Problem
A starvation free implementation of the classical Reader-Writer's Problem.

## Classical Problem :
It refers to a situation where a file is shared among multiple users.It can also be a shared memory location. There are two types of users 
1. Writers - They edit the file 
2. Readers - They read the file but do not make any changes.
<hr>
For data consistency, the following must be ensured : -
a) If the writer is editng the file, none of the readers should be able to access it. Similiary, if one or multiple readers are already reading the file, the writer should not be able to edit it. <br>
b) On the otherhand, multiple readers can access the file simultaneously.

There are two classical implementations of the Reader Writer's Problem.
1. SET1 - Prioritizes the readers where the writers may starve.
Shared Variables and Semaphores:-
```cpp
Semaphore wrt; // To ensure exclusion between writers and writers
Semaphore mutex; //To prevent interleaving of readcount++ and readcount-- operations.

```
The pseudocodes for the processes are as follows:-
```cpp
Writer_Process()
{
    wait(wrt);
    /*
    ...
    Critical Section writing is performed
    ...
    */
    signal(wrt)
}

Reader_Process()
{
    wait(mutex);
    readcount++;
    if (readcount == 1) wait(wrt);
    signal(mutex);
    /*
    ...
    reading is performed
    ...
    */
    wait(mutex);
    readcount--;
    if (readcount == 0) signal(wrt);
    signal(mutex):
}
```
Here the mutex just ensures atomicity of the increment and decrement operations on the readcount variable.
#### Mutual Exclusion:
The wrt semaphore is changed by the reader processes only when a there are no readers and a reader process starts executing. It is signaled only when all the reader processes have completed their execution. The writer can execute only when the readcount is 0, i.e. the number of readers currently reading the file is 0. 
Readers wait for wrt thus they cannot execute while writer has possession of wrt. Thus mutual exclusion is ensured.

### Starvation:
In this solution there is a chance that the writers may starve. If a reader is executing and there are a large number of reader processes, the writer can write only after all the reader processes have finished execution. 


## Starvation Free Solution:
```cpp
//Shared Semaphores
Semaphore in_mutex = 1;
Semaphore out_mutex = 1; 
Semaphore rw = 0;
bool writer_waiting = false;
int readers_request = 0; //represents the number of readers that have started executing 
int readers_granted_request= 0; //represents the number of readers that have completed the reading

Reader-Process
{
    while(true)
    {
        wait(in_mutex);
        wait(out_mutex);
        readers_request++;
        signal(out_mutex);
        signal(in_mutex);
        /*q
            Critical section
            which needs to be exclusive 
            writing
        */
       wait(out_mutex);
       readers_granted_request++;
       if(writer_waiting && readers_request == readers_granted_request)
       {
            signal(rw);
       }
       signal(out_mutex);
    }
}
```

```cpp
Writer-Process
{
    while(true)
    {
        wait(in_mutex);
        wait(out_mutex);
        if(readers_request == readers_granted_request)//we will start writing
        {
            signal(out_mutex);
        }else{//we will wait
            writer_waiting = true;
            signal(out_mutex);
            wait(rw);
            writer_waiting = false;
        }
        /*
            Critical section
            which needs to be exclusive 
            reading
        */
       signal(in_mutex);
    }
}
```
## Pseudocode Explanation
#### Shared Resources (and initialisation)
```cpp
The file which is being read/ write
Semaphore CanRead = 1;
Semaphore Shared_Vars = 1;
Semaphore rw = 0;
bool writer_waiting = false;
int readers_request = 0;
int readers_granted_request= 0;
```
Readers process :
```cpp
Reader_Process
{
    while(true)
    {
        wait(CanRead);
        readers_request++;
        signal(CanRead);
        /*q
            Critical section
            which needs to be exclusive 
            writing
        */
       wait(Shared_Vars);
       readers_granted_request++;
       if(writer_waiting && readers_request == readers_granted_request)
       {
            signal(rw);
       }
       signal(Shared_Vars);
    }
}
```
Writer's process :
```cpp
Writer_Process
{
    while(true)
    {
        wait(CanRead);
        wait(Shared_Vars);
        if(readers_request == readers_granted_request)//we will start writing
        {
            signal(Shared_Vars);
        }else{//we will wait
            writer_waiting = true;
            signal(Shared_Vars);
            wait(rw);
            writer_waiting = false;
        }
        /*
            Critical section
            which needs to be exclusive 
            reading
        */
       signal(CanRead);
    }
}
```


#### READERS LOGIC
The 'CanRead' binary semaphores ensures mutual exclusion for reader processes from writer processes. <br>
The writer waits for acquiring the CanRead semaphore before starting execution and signals it only after the critical section is exited. <br>
The reader waits for the CanRead semaphore, thus it cannot execute while the writer process is in its critical section. <br>
Readers signal the CanRead semaphore right after increasing the value of reader_request variable. <br>
This allows other Readers to read the file.

#### WRITER LOGIC
Once the CanRead is acquired, there are two possible cases,
All readers that started the process have completed it, or some readers are still reading. <br>
In the first case, the if condition is true and the critical section is executed.<br>
In the second case, the writer enters the else condition and waits for the rw signal which is set by the reader processes only when all the readers that have started the reading process have finished executing it.<br>
NOTE: During this time CanRead is set to 0 thus, no new read processes will be started, thus this wait time is finite. The writer waits only for those read processes that are already executing. <br>




