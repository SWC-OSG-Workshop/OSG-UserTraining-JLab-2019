---
layout: lesson
root: ../..
title: 	Job Scheduling with HTCondor  
---
<!-- <div class="objectives" markdown="1">

#### Objectives
*   Learn how to submit HTCondor Jobs.   
*   Learn how to monitor the running Jobs.    
</div> -->

## Overview 
In this section, we will learn the basics of HTCondor for submitting and monitoring jobs. The jobs are submitted through the OSG Connect login node. The submitted jobs are executed on the remote worker node(s) and the outputs are transfered back to the login node. 

![fig 1](https://raw.githubusercontent.com/OSGConnect/osg-ahm-17/novice/DHTC/Images/jobSubmit.png)

## Login to OSG Connect

If you are still logged in from the previous lesson, you can continue to the next section. Otherwise, please login using SSH:

```
# username is your username
$ ssh username@training.osgconnect.net  
# enter your password
password:                            
```

You may receive a warning such as:
```
The authenticity of host 'training.osgconnect.net (192.170.227.119)' can't be established.
RSA key fingerprint is SHA256:KRH0+kF1V5kNookplCt2f+lH4dKaZLowKbEevNnVmKY.
Are you sure you want to continue connecting (yes/no)?
```

This is normal when connecting to a new host for the first time. If prompted, type `yes`.

## Job execution script

Using the `tutorial` command, we will get the relevant example files. Run the `quickstart` tutorial:

```
# creates a directory "tutorial-quickstart".
$ tutorial quickstart 
# relevant script and input files are inside this directory
$ cd ~/tutorial-quickstart 
```

We will look at two files in detail: `short.sh` and `tutorial01.submit`. You can use your favorite text editor, we will be using `cat`:

```
$ cat short.sh
```

```
#!/bin/bash
# short.sh: a short discovery job
printf "Start time: "; /bin/date
printf "Job is running on node: "; /bin/hostname
printf "Job running as user: "; /usr/bin/id
printf "Job is running in directory: "; /bin/pwd
echo
echo "Working hard..."
sleep 10
echo "Science complete!"
```

Now, make the script executable.

```
$ chmod +x short.sh 
```

Since we used the tutorial command, all files are already in your workspace. Run the job locally when setting up a new job type, it is important to test your job outside of HTCondor before submitting into the Open Science Grid. 

```
$ ./short.sh
```

```
Start time: Mon Mar  6 00:08:06 CST 2017
Job is running on node: training.osgconnect.net
Job running as user: uid=46628(username) gid=46628(username) groups=46628(username),400(condor),401(ciconnect-staff),1000(users)
Job is running in directory: /tmp/Test/tutorial-quickstart

Working hard...
Science complete!
```

## Job submission file
The HTCondor submit file describes the job requirements, how to execute the program, and what (small) input/output data needs to be transfered.

```
$ cat tutorial01.submit
```

```
# The UNIVERSE defines an execution environment. You will almost always use "vanilla". 
Universe = vanilla 

# EXECUTABLE is the program your job will run. It's often useful to create a shell script to "wrap" your actual work. 
Executable = short.sh 

# ERROR and OUTPUT are the error and output channels from your job that HTCondor returns from the remote host.
Error = job.error
Output = job.output

# The LOG file is where HTCondor places information about your  job's status, success, and resource consumption. 
Log = job.log

# Queue is the "start button" - it launches any jobs that have been  specified thus far. 
Queue 1
```

## Job submission
Submit the job using `condor_submit`:

```
$ condor_submit tutorial01.submit
Submitting job(s).
1 job(s) submitted to cluster 1144.
```

## Job status

The `condor_q` command tells the status of currently running jobs. Please note, that the `condor_q` command line interface has changed in recent HTCondor versions, and in this tutorial we are using the new version.

```
$ condor_q

-- Schedd: training.osgconnect.net : <192.170.227.119:9618?... @ 06/30/17 15:17:51
OWNER    BATCH_NAME       SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
username CMD: short.sh   6/30 15:17      _      _      1      1 1872.0

Total for query: 1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended 
Total for username: 1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended 
Total for all users: 1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended
```

This new output format "batches" similar jobs together. If you want to see each individual job, use the `-nobatch` option:

```
$ condor_q -nobatch

-- Schedd: training.osgconnect.net : <192.170.227.119:9618?... @ 06/30/17 15:19:21
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD
1873.0   rynge           6/30 15:19   0+00:00:00 I  0    0.0 short.sh

Total for query: 1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended 
Total for rynge: 1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended 
Total for all users: 1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended
```

If you want to see all jobs running on the system, use `condor_q -allusers`.

You can also get status on a specific job cluster:

```
$ condor_q -nobatch 1144.0 

-- Schedd: training.osgconnect.net : <192.170.227.119:9419?...
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
1144.0   username       3/6  00:17   0+00:00:00 I  0   0.0  short.sh

1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended
```

Note the ST (state) column. Your job will be in the `I` state (idle) if it hasn't started yet. If it's currently scheduled and running, it will have state `R` (running). If it has completed already, it will not appear in `condor_q`.

Let's wait for your job to finish – that is, for `condor_q` not to show the job in its output. A useful tool for this is watch – it runs a program repeatedly, letting you see how the output differs at fixed time intervals. Let's submit the job again, and watch condor_q output at two-second intervals:

```
$ condor_submit tutorial01.submit
Submitting job(s). 
1 job(s) submitted to cluster 1145
$ watch -n2 condor_q username 
```

When your job has completed, it will disappear from the list.  To close watch, hold down Ctrl and press C.

## Job history

Once your job has finished, you can get information about its execution from the `condor_history` command:

```
$ condor_history 1144 
 ID      OWNER            SUBMITTED     RUN_TIME ST   COMPLETED CMD
 1144.0   username          3/6 09:46   0+00:00:12 C   3/6 09:46 /home/username/
```

You can see much more information about your job's final status using the `-long` option, i.e. `condor_history -long <job_id>`


## Job output

Once your job has finished, you can look at the files that HTCondor has returned to the working directory. If everything was successful, it should have returned three files:

x

Read the output file. It should be something like this:

```
$ cat job.output
Start time: Mon Mar  6 00:08:06 CST 2017
Job is running on node: training.osgconnect.netJob running as user: uid=46628(username) gid=46628(username) groups=46628(username),400(condor),401(ciconnect-staff),1000(users)
Job is running in directory: /tmp/Test/tutorial-quickstart

Working hard...
Science complete!
``` 

<!-- <div class="keypoints" markdown="1">

#### Key Points
*   HTCondor shedules and monitors your Jobs. 
*   To submit a job to HTCondor, the user need to prepare the Job execution and Job Submission scripts. 
*   *condor_submit* - HTCondor's job submission command.     
*   *condor_q* - HTCondor's job monitoring command.     
*   *condor_rm* - HTCondor's job removal command.     
</div> -->
