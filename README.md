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
2. SET2 - Prioritizes the writers where the readers may starve.

## Starvation Free Solution:
``` 'c_cpp'
//Shared Semaphores
Semaphore in_mutex = 1;
Semaphore out_mutex = 1;
Semaphore rw = 0;
bool writer_waiting = false;
int readers_request = 0;
int readers_granted_request= 0;

Reader-Process
{
    while(true)
    {
        wait(in_mutex);
        readers_request++;
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
#### Shared Resources
The file which is being read/ write
Semaphore CanRead = 1;
Semaphore Shared_Vars = 1;
Semaphore rw = 0;
bool writer_waiting = false;
int readers_request = 0;
int readers_granted_request= 0;


#### READERS LOGIC
The 'CanRead' binary semaphores ensures mutual exclusion for reader processes from writer processes. <br>
The writer waits for acquiring the CanRead semaphore before starting execution and signals it only after the critical section is exited. <br>
The reader waits for the can read process, thus it cannot execute while the writer process is in its critical section. <br>
Readers signal the CanRead semaphore right after increasing the value of reader_request variable. <br>
This allows other Readers to read the file.

#### WRITER LOGIC
Once the CanRead is acquired, there are two possible cases,
All readers that started the process have completed it, or some readers are still reading. <br>
In the first case, the if condition is true and the critical section is executed.<br>
In the second case, the writer enters the else condition and waits for the rw signal which is set by the reader processes only when all the readers that have started the reading process have finished executing it.<br>
NOTE: During this time CanRead is set to 0 thus, no new read processes will be started, thus this wait time is finite. <br>




