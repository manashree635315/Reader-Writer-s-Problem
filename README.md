# Reader Writer's Problem
A starvation free implementation of the classical Reader-Writer's Problem.

## Classical Problem :
It refers to a situation where a file is shared among multiple users.It can also be a shared memory location. There are two types of users 
1. Writers - They edit the file 
2. Readers - They read the file but do not make any changes.
<hr>
For data consistency, the following must be ensured : -
* If the writer is editng the file, none of the readers should be able to access it. Similiary, if one or multiple readers are already reading the file, the writer should not be able to edit it.
* On the otherhand, multiple readers can access the file simultaneously.

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



