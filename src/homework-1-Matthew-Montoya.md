# CS 4375 - Operating Systems

## Homework 1: Chapter 4 Questions

### Author: Matthew S Montoya

</br>

**1. Run the program with the following flags: ```./process-run.py -l 5:100,5:100```. What should the CPU utilization be (e.g., the percent of time the CPU is in use?) Why do you know this? Use the ```-c``` and ```-p``` flags to see if you were right.**

CPU utilization should be at 100%. We know this because we’re passing through a parameter ```5:100```, which passes five instructions running the CPU at 100%. ```Running ./process-run.py -l 5:100,5:100 -c -p``` confirms this prediction, as shown below.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 5:100,5:100 -c -p
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2     RUN:cpu      READY          1            
  3     RUN:cpu      READY          1            
  4     RUN:cpu      READY          1            
  5     RUN:cpu      READY          1            
  6        DONE    RUN:cpu          1            
  7        DONE    RUN:cpu          1            
  8        DONE    RUN:cpu          1            
  9        DONE    RUN:cpu          1            
 10        DONE    RUN:cpu          1            

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```

</br>

**2. Now run with these flags: ```./process-run.py -l 4:100,1:0```. These flags specify one process with 4 instructions (all to use the CPU), and one that simply issues an I/O and waits for it to be done. How long does it take to complete both processes? Use ```-c``` and ```-p``` to find out if you were right.**

It should take 10 clock ticks because we’re specifying five instructions, with four instructions requiring one clock tick (CPU), IO requiring five ticks, and lastly, the "done" tick. Running ```./process-run.py -l 4:100,1:0 -c -p``` confirms this prediction, as shown below.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 4:100,1:0 -c -p
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2     RUN:cpu      READY          1            
  3     RUN:cpu      READY          1            
  4     RUN:cpu      READY          1            
  5        DONE     RUN:io          1            
  6        DONE    WAITING                     1 
  7        DONE    WAITING                     1 
  8        DONE    WAITING                     1 
  9        DONE    WAITING                     1 
 10*       DONE       DONE                       

Stats: Total Time 10
Stats: CPU Busy 5 (50.00%)
Stats: IO Busy  4 (40.00%)
```

</br>

**3. Now switch the order of the processes: ```./process-run.py -l 1:0,4:100```. What happens now? Does switching the order matter? Why? (As always, use ```-c``` and ```-p``` to see if you were right)**

Switching the order of the process matters because only one instruction is to use the CPU (which requires one clock) & four instructions that issue an I/O (and require five clock ticks). In this case, we should expect six clock ticks. Running ```process-run.py -l 1:0,4:100 -c -p```  confirms this prediction.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 1:0,4:100 -c -p
Time     PID: 0     PID: 1        CPU        IOs 
  1      RUN:io      READY          1            
  2     WAITING    RUN:cpu          1          1 
  3     WAITING    RUN:cpu          1          1 
  4     WAITING    RUN:cpu          1          1 
  5     WAITING    RUN:cpu          1          1 
  6*       DONE       DONE                       

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```

</br>

**4. We’ll now explore some of the other flags. One important flag is ```-S```, which determines how the system reacts when a process issues an I/O. With the flag set to ```SWITCH_ON_END```, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes, one doing I/O and the other doing CPU work? (```-l 1:0,4:100 -c -S SWITCH ON END```)**

When we run the process doing I/O, it takes nine clock ticks to execute, versus the previous six clock ticks. This occurs because we are now waiting for the previous process to completely execute. In this process, the CPU was busy 55.56% of the time. When we run the process doing CPU work, it takes 10 clock ticks to execute, which is the same output from question #2. In this process, the CPU was busy no more than 50% of the time. Additionally, we notice that the two processes cannot run concurrently while in the I/O.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_END
Time     PID: 0     PID: 1        CPU        IOs 
  1      RUN:io      READY          1            
  2     WAITING      READY                     1 
  3     WAITING      READY                     1 
  4     WAITING      READY                     1 
  5     WAITING      READY                     1 
  6*       DONE    RUN:cpu          1            
  7        DONE    RUN:cpu          1            
  8        DONE    RUN:cpu          1            
  9        DONE    RUN:cpu          1            

Stats: Total Time 9
Stats: CPU Busy 5 (55.56%)
Stats: IO Busy  4 (44.44%)
```

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 4:100,1:0 -c -p -S SWITCH_ON_END
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2     RUN:cpu      READY          1            
  3     RUN:cpu      READY          1            
  4     RUN:cpu      READY          1            
  5        DONE     RUN:io          1            
  6        DONE    WAITING                     1 
  7        DONE    WAITING                     1 
  8        DONE    WAITING                     1 
  9        DONE    WAITING                     1 
 10*       DONE       DONE                       

Stats: Total Time 10
Stats: CPU Busy 5 (50.00%)
Stats: IO Busy  4 (40.00%)
```

</br>

**5. Now, run the same processes, but with the switching behavior set to switch to another process whenever one is ```WAITING``` for I/O (```-l 1:0,4:100 -c -S SWITCH_ON_IO```). What happens now? Use ```-c``` and ```-p``` to confirm that you are right.**

When we run the process doing I/O, we reduce the time back to 6 clock ticks, however the CPU increases utilization to upwards of 84%. When running the process using the CPU, we an expected 10 ticks. With CPU utilization reaching 50%. Running ```process-run.py - l 1:0,4:100 -c -p -S SWITCH_ON_IO``` confirms this prediction. This is because when we pass through the ```SWITCH_ON_IO``` parameter, PID:1 can run an instruction set while ```PID:0``` is in I/O.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_IO
Time     PID: 0     PID: 1        CPU        IOs 
  1      RUN:io      READY          1            
  2     WAITING    RUN:cpu          1          1 
  3     WAITING    RUN:cpu          1          1 
  4     WAITING    RUN:cpu          1          1 
  5     WAITING    RUN:cpu          1          1 
  6*       DONE       DONE                       

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 4:100,1:0 -c -p -S SWITCH_ON_IO
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2     RUN:cpu      READY          1            
  3     RUN:cpu      READY          1            
  4     RUN:cpu      READY          1            
  5        DONE     RUN:io          1            
  6        DONE    WAITING                     1 
  7        DONE    WAITING                     1 
  8        DONE    WAITING                     1 
  9        DONE    WAITING                     1 
 10*       DONE       DONE                       

Stats: Total Time 10
Stats: CPU Busy 5 (50.00%)
Stats: IO Busy  4 (40.00%)
```

</br>

**6. One other important behavior is what to do when an I/O completes. With ```-I IO_RUN_LATER```, when an I/O completes, the pro- c 2014, ARPACI-DUSSEAU THREE EASY PIECES 12 THE ABSTRACTION: THE PROCESS cess that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when you run this combination of processes? (```./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p```) Are system resources being effectively utilized?**

When we run the process, passing through the ```-I IO_RUN_LATER``` parameter, we notice the time takes 26 clock ticks. Despite the increase in the time compared to the previous program executions, this time increase comes from passing more parameters, ```3:0,5:100,5:100,5:100```. Analyzing the output of the process, we notice that the system resources are effectively being utilized. For example, while ```PID:0``` is in I/O, ```PID:1```, ```PID:2```, and ```PID:3``` continue executing on the CPU, with each process in actively ```WAITING``` until the previous process is complete.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
Time     PID: 0     PID: 1     PID: 2     PID: 3        CPU        IOs 
  1      RUN:io      READY      READY      READY          1            
  2     WAITING    RUN:cpu      READY      READY          1          1 
  3     WAITING    RUN:cpu      READY      READY          1          1 
  4     WAITING    RUN:cpu      READY      READY          1          1 
  5     WAITING    RUN:cpu      READY      READY          1          1 
  6*      READY    RUN:cpu      READY      READY          1            
  7       READY       DONE    RUN:cpu      READY          1            
  8       READY       DONE    RUN:cpu      READY          1            
  9       READY       DONE    RUN:cpu      READY          1            
 10       READY       DONE    RUN:cpu      READY          1            
 11       READY       DONE    RUN:cpu      READY          1            
 12       READY       DONE       DONE    RUN:cpu          1            
 13       READY       DONE       DONE    RUN:cpu          1            
 14       READY       DONE       DONE    RUN:cpu          1            
 15       READY       DONE       DONE    RUN:cpu          1            
 16       READY       DONE       DONE    RUN:cpu          1            
 17      RUN:io       DONE       DONE       DONE          1            
 18     WAITING       DONE       DONE       DONE                     1 
 19     WAITING       DONE       DONE       DONE                     1 
 20     WAITING       DONE       DONE       DONE                     1 
 21     WAITING       DONE       DONE       DONE                     1 
 22*     RUN:io       DONE       DONE       DONE          1            
 23     WAITING       DONE       DONE       DONE                     1 
 24     WAITING       DONE       DONE       DONE                     1 
 25     WAITING       DONE       DONE       DONE                     1 
 26     WAITING       DONE       DONE       DONE                     1 
 27*       DONE       DONE       DONE       DONE                       

Stats: Total Time 27
Stats: CPU Busy 18 (66.67%)
Stats: IO Busy  12 (44.44%)
```

</br>

**7. Now run the same processes, but with ```-I IO_RUN_IMMEDIATE``` set, which immediately runs the process that issued the I/O. How does this behavior differ? Why might running a process that just completed an I/O again be a good idea?**

When running this process, we notice this behaves differently from the previous process. We notice the time reduced to 18 clock ticks, which can be attributed to ```-I IO_RUN_IMMEDIATE``` forcing a run of the process that issued the I/O, as noted in the question. This is because ```IO_RUN_LATER``` switching incurs time, based on behavior of the process-switching. Running a process that just completed an I/O again is a good idea because this allows for less delay, making the overall system faster and more responsive.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p
Time     PID: 0     PID: 1     PID: 2     PID: 3        CPU        IOs 
  1      RUN:io      READY      READY      READY          1            
  2     WAITING    RUN:cpu      READY      READY          1          1 
  3     WAITING    RUN:cpu      READY      READY          1          1 
  4     WAITING    RUN:cpu      READY      READY          1          1 
  5     WAITING    RUN:cpu      READY      READY          1          1 
  6*     RUN:io      READY      READY      READY          1            
  7     WAITING    RUN:cpu      READY      READY          1          1 
  8     WAITING       DONE    RUN:cpu      READY          1          1 
  9     WAITING       DONE    RUN:cpu      READY          1          1 
 10     WAITING       DONE    RUN:cpu      READY          1          1 
 11*     RUN:io       DONE      READY      READY          1            
 12     WAITING       DONE    RUN:cpu      READY          1          1 
 13     WAITING       DONE    RUN:cpu      READY          1          1 
 14     WAITING       DONE       DONE    RUN:cpu          1          1 
 15     WAITING       DONE       DONE    RUN:cpu          1          1 
 16*       DONE       DONE       DONE    RUN:cpu          1            
 17        DONE       DONE       DONE    RUN:cpu          1            
 18        DONE       DONE       DONE    RUN:cpu          1            

Stats: Total Time 18
Stats: CPU Busy 18 (100.00%)
Stats: IO Busy  12 (66.67%)
```

</br>

**8. Now run with some randomly generated processes, e.g., ```-s 1 -l 3:50,3:50```, ```-s 2 -l 3:50,3:50```, ```-s 3 -l 3:50,3:50```. See if you can predict how the trace will turn out. What happens when you use ```-I_IO_RUN_IMMEDIATE``` vs. ```-I IO_RUN_LATER```? What happens when you use ```-S SWITCH_ON_IO``` vs. ```-S SWITCH_ON_END```?**

When running ```-S SWITCH_ON_IO``` vs. ```-S SWITCH_ON_END``` flags, we observe that ```SWITCH_ON_END``` uses less computational resources than ```SWITCH_ON_IO```. In doing so, we see an increase in the time it takes to run the process. This confirms our predication. This time increase can be combatted by using flags that processes to execute while another process in in I/O, i.e. the ```-I IO_RUN_IMMEDIATE``` flag.

When running ```-I IO_RUN_IMMEDIATE``` vs. ```-I IO_RUN_LATER```, we notice nearly identical output from the terminal. This output confirms that by executing a process while another process in in I/O, we can reduce the total time it takes to execute the processes. Moreover, we notice that a similarity between the two flags; the total time was the same. This is something that can be attributed to a commonality ```IO_RUN_LATER``` shares with IO_RUN_IMMEDIATE in this particular execution: that the time when ```IO_RUN_LATER``` would naturally switch to the process also coincided with the time that ```IO_RUN_IMMEDIATE``` called for the process to switch, that is right away.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -s 1 -l 3:50,3:50 -S SWITCH_ON_IO -c -p
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2      RUN:io      READY          1            
  3     WAITING    RUN:cpu          1          1 
  4     WAITING    RUN:cpu          1          1 
  5     WAITING    RUN:cpu          1          1 
  6     WAITING       DONE                     1 
  7*     RUN:io       DONE          1            
  8     WAITING       DONE                     1 
  9     WAITING       DONE                     1 
 10     WAITING       DONE                     1 
 11     WAITING       DONE                     1 
 12*       DONE       DONE                       

Stats: Total Time 12
Stats: CPU Busy 6 (50.00%)
Stats: IO Busy  8 (66.67%)

```

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -s 1 -l 3:50,3:50 -S SWITCH_ON_END -c -p
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2      RUN:io      READY          1            
  3     WAITING      READY                     1 
  4     WAITING      READY                     1 
  5     WAITING      READY                     1 
  6     WAITING      READY                     1 
  7*     RUN:io      READY          1            
  8     WAITING      READY                     1 
  9     WAITING      READY                     1 
 10     WAITING      READY                     1 
 11     WAITING      READY                     1 
 12*       DONE    RUN:cpu          1            
 13        DONE    RUN:cpu          1            
 14        DONE    RUN:cpu          1            

Stats: Total Time 14
Stats: CPU Busy 6 (42.86%)
Stats: IO Busy  8 (57.14%)

```

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_IMMEDIATE -c -p
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2      RUN:io      READY          1            
  3     WAITING    RUN:cpu          1          1 
  4     WAITING    RUN:cpu          1          1 
  5     WAITING    RUN:cpu          1          1 
  6     WAITING       DONE                     1 
  7*     RUN:io       DONE          1            
  8     WAITING       DONE                     1 
  9     WAITING       DONE                     1 
 10     WAITING       DONE                     1 
 11     WAITING       DONE                     1 
 12*       DONE       DONE                       

Stats: Total Time 12
Stats: CPU Busy 6 (50.00%)
Stats: IO Busy  8 (66.67%)

```

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_LATER -c -p
Time     PID: 0     PID: 1        CPU        IOs 
  1     RUN:cpu      READY          1            
  2      RUN:io      READY          1            
  3     WAITING    RUN:cpu          1          1 
  4     WAITING    RUN:cpu          1          1 
  5     WAITING    RUN:cpu          1          1 
  6     WAITING       DONE                     1 
  7*     RUN:io       DONE          1            
  8     WAITING       DONE                     1 
  9     WAITING       DONE                     1 
 10     WAITING       DONE                     1 
 11     WAITING       DONE                     1 
 12*       DONE       DONE                       

Stats: Total Time 12
Stats: CPU Busy 6 (50.00%)
Stats: IO Busy  8 (66.67%)

```

</br>

**9. Suppose we have two interactive processes with 10 instructions and 10% CPU use. We have two non-interactive processes, 10 instructions, 90% CPU. Run a simulation that minimizes the total time required for these four processes. You will need to set some of the flags suggested in questions 1-8. ```./process-run.py -l 10:10,10:10,10:90,10:90``` Turn in the command line arguments and a screen capture of the output. Explain why you believe your result optimizes runtime.**

When running the ```./process-run.py -l 10:10,10:10,10:90,10:90 -c -p``` as provided in the problem statement, we notice a total time of 68 clock ticks. However, as we learned in problems #7 & #8, by including the ```IO_RUN_IMMEDIATE``` and ```SWITCH_ON_IO``` flags, we can reduce the runtime, as it does here. I believe this optimizes runtime because ```IO_RUN_IMMEDIATE``` forces the execution of the next process while there is another process in I/O. In doing so, this optimizes the runtime down from 68 clock ticks to 52 clock ticks, as seen by the output below.

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 10:10,10:10,10:90,10:90 -c -p
Time     PID: 0     PID: 1     PID: 2     PID: 3        CPU        IOs 
  1      RUN:io      READY      READY      READY          1            
  2     WAITING     RUN:io      READY      READY          1          1 
  3     WAITING    WAITING    RUN:cpu      READY          1          2 
  4     WAITING    WAITING    RUN:cpu      READY          1          2 
  5     WAITING    WAITING    RUN:cpu      READY          1          2 
  6*      READY    WAITING    RUN:cpu      READY          1          1 
  7*      READY      READY    RUN:cpu      READY          1            
  8       READY      READY    RUN:cpu      READY          1            
  9       READY      READY    RUN:cpu      READY          1            
 10       READY      READY    RUN:cpu      READY          1            
 11       READY      READY     RUN:io      READY          1            
 12       READY      READY    WAITING    RUN:cpu          1          1 
 13       READY      READY    WAITING    RUN:cpu          1          1 
 14       READY      READY    WAITING    RUN:cpu          1          1 
 15       READY      READY    WAITING    RUN:cpu          1          1 
 16*      READY      READY      READY    RUN:cpu          1            
 17       READY      READY      READY    RUN:cpu          1            
 18       READY      READY      READY    RUN:cpu          1            
 19       READY      READY      READY    RUN:cpu          1            
 20       READY      READY      READY    RUN:cpu          1            
 21       READY      READY      READY    RUN:cpu          1            
 22      RUN:io      READY      READY       DONE          1            
 23     WAITING     RUN:io      READY       DONE          1          1 
 24     WAITING    WAITING     RUN:io       DONE          1          2 
 25     WAITING    WAITING    WAITING       DONE                     3 
 26     WAITING    WAITING    WAITING       DONE                     3 
 27*     RUN:io    WAITING    WAITING       DONE          1          2 
 28*    WAITING     RUN:io    WAITING       DONE          1          2 
 29*    WAITING    WAITING       DONE       DONE                     2 
 30     WAITING    WAITING       DONE       DONE                     2 
 31     WAITING    WAITING       DONE       DONE                     2 
 32*     RUN:io    WAITING       DONE       DONE          1          1 
 33*    WAITING     RUN:io       DONE       DONE          1          1 
 34     WAITING    WAITING       DONE       DONE                     2 
 35     WAITING    WAITING       DONE       DONE                     2 
 36     WAITING    WAITING       DONE       DONE                     2 
 37*     RUN:io    WAITING       DONE       DONE          1          1 
 38*    WAITING     RUN:io       DONE       DONE          1          1 
 39     WAITING    WAITING       DONE       DONE                     2 
 40     WAITING    WAITING       DONE       DONE                     2 
 41     WAITING    WAITING       DONE       DONE                     2 
 42*     RUN:io    WAITING       DONE       DONE          1          1 
 43*    WAITING     RUN:io       DONE       DONE          1          1 
 44     WAITING    WAITING       DONE       DONE                     2 
 45     WAITING    WAITING       DONE       DONE                     2 
 46     WAITING    WAITING       DONE       DONE                     2 
 47*     RUN:io    WAITING       DONE       DONE          1          1 
 48*    WAITING     RUN:io       DONE       DONE          1          1 
 49     WAITING    WAITING       DONE       DONE                     2 
 50     WAITING    WAITING       DONE       DONE                     2 
 51     WAITING    WAITING       DONE       DONE                     2 
 52*     RUN:io    WAITING       DONE       DONE          1          1 
 53*    WAITING     RUN:io       DONE       DONE          1          1 
 54     WAITING    WAITING       DONE       DONE                     2 
 55     WAITING    WAITING       DONE       DONE                     2 
 56     WAITING    WAITING       DONE       DONE                     2 
 57*     RUN:io    WAITING       DONE       DONE          1          1 
 58*    WAITING     RUN:io       DONE       DONE          1          1 
 59     WAITING    WAITING       DONE       DONE                     2 
 60     WAITING    WAITING       DONE       DONE                     2 
 61     WAITING    WAITING       DONE       DONE                     2 
 62*     RUN:io    WAITING       DONE       DONE          1          1 
 63*    WAITING     RUN:io       DONE       DONE          1          1 
 64     WAITING    WAITING       DONE       DONE                     2 
 65     WAITING    WAITING       DONE       DONE                     2 
 66     WAITING    WAITING       DONE       DONE                     2 
 67*       DONE    WAITING       DONE       DONE                     1 
 68*       DONE       DONE       DONE       DONE                       

Stats: Total Time 68
Stats: CPU Busy 40 (58.82%)
Stats: IO Busy  54 (79.41%)
```

```text
mmedina-a@Leviathan ~/Downloads> ./process-run.py -l 10:10,10:10,10:90,10:90 -I IO_RUN_IMMEDIATE -S SWITCH_ON_IO -c -p
Time     PID: 0     PID: 1     PID: 2     PID: 3        CPU        IOs 
  1      RUN:io      READY      READY      READY          1            
  2     WAITING     RUN:io      READY      READY          1          1 
  3     WAITING    WAITING    RUN:cpu      READY          1          2 
  4     WAITING    WAITING    RUN:cpu      READY          1          2 
  5     WAITING    WAITING    RUN:cpu      READY          1          2 
  6*     RUN:io    WAITING      READY      READY          1          1 
  7*    WAITING     RUN:io      READY      READY          1          1 
  8     WAITING    WAITING    RUN:cpu      READY          1          2 
  9     WAITING    WAITING    RUN:cpu      READY          1          2 
 10     WAITING    WAITING    RUN:cpu      READY          1          2 
 11*     RUN:io    WAITING      READY      READY          1          1 
 12*    WAITING     RUN:io      READY      READY          1          1 
 13     WAITING    WAITING    RUN:cpu      READY          1          2 
 14     WAITING    WAITING    RUN:cpu      READY          1          2 
 15     WAITING    WAITING     RUN:io      READY          1          2 
 16*     RUN:io    WAITING    WAITING      READY          1          2 
 17*    WAITING     RUN:io    WAITING      READY          1          2 
 18     WAITING    WAITING    WAITING    RUN:cpu          1          3 
 19     WAITING    WAITING    WAITING    RUN:cpu          1          3 
 20*    WAITING    WAITING     RUN:io      READY          1          2 
 21*     RUN:io    WAITING    WAITING      READY          1          2 
 22*    WAITING     RUN:io    WAITING      READY          1          2 
 23     WAITING    WAITING    WAITING    RUN:cpu          1          3 
 24     WAITING    WAITING    WAITING    RUN:cpu          1          3 
 25*    WAITING    WAITING       DONE    RUN:cpu          1          2 
 26*     RUN:io    WAITING       DONE      READY          1          1 
 27*    WAITING     RUN:io       DONE      READY          1          1 
 28     WAITING    WAITING       DONE    RUN:cpu          1          2 
 29     WAITING    WAITING       DONE    RUN:cpu          1          2 
 30     WAITING    WAITING       DONE    RUN:cpu          1          2 
 31*     RUN:io    WAITING       DONE      READY          1          1 
 32*    WAITING     RUN:io       DONE      READY          1          1 
 33     WAITING    WAITING       DONE    RUN:cpu          1          2 
 34     WAITING    WAITING       DONE    RUN:cpu          1          2 
 35     WAITING    WAITING       DONE       DONE                     2 
 36*     RUN:io    WAITING       DONE       DONE          1          1 
 37*    WAITING     RUN:io       DONE       DONE          1          1 
 38     WAITING    WAITING       DONE       DONE                     2 
 39     WAITING    WAITING       DONE       DONE                     2 
 40     WAITING    WAITING       DONE       DONE                     2 
 41*     RUN:io    WAITING       DONE       DONE          1          1 
 42*    WAITING     RUN:io       DONE       DONE          1          1 
 43     WAITING    WAITING       DONE       DONE                     2 
 44     WAITING    WAITING       DONE       DONE                     2 
 45     WAITING    WAITING       DONE       DONE                     2 
 46*     RUN:io    WAITING       DONE       DONE          1          1 
 47*    WAITING     RUN:io       DONE       DONE          1          1 
 48     WAITING    WAITING       DONE       DONE                     2 
 49     WAITING    WAITING       DONE       DONE                     2 
 50     WAITING    WAITING       DONE       DONE                     2 
 51*       DONE    WAITING       DONE       DONE                     1 
 52*       DONE       DONE       DONE       DONE                       

Stats: Total Time 52
Stats: CPU Busy 40 (76.92%)
Stats: IO Busy  50 (96.15%)
```
