---
title: "Working on an HPC"
teaching: 15
exercises: 15
questions:
- "What will we be learning?"
objectives:
- "Review schedule"
keypoints:
- "This is not a lecture"
---

## Logging in 
We will be working with the Swinburne HPC cluster called OzStar, you should have an account, and be a member of the group `oz983`.
See the [setup]({{page.root}}{% link _episodes/01_setup.md %}#an-account-on-ozstar) page if you need more information.

To connect to a login node you should use ssh:
```bash
ssh <user>@ozstar.swin.edu.au
```
You will be asked for a password each time you connect.
If you would prefer to work with ssh keys for password-less login, [this tutorial](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/) is quite helpful.

Upon login you'll be greeted with a message of the day:

```output
-----------------------------------------------------------------------------
Website: http://supercomputing.swin.edu.au
------------------------------------------------------------------------------
Please make sure you keep up-to-date with the usage policies outlined within
the website, noting that with the exception of your home directory, NO data on
any OzSTAR filesystems is backed up. Repeat: NOT backed up.

Home directories have a strict 10GB quota. All large data files, such as job
output, should be written to the dagg filesystem at /fred/<project>.

You can use the 'quota' command to check usage on both filesystems.
------------------------------------------------------------------------------
The OzStar cluster login nodes farnarkle1 and 2 are available to use as
interactive nodes for debugging and running small jobs. Login sessions
are limited to 4 cores per user. Login nodes may be rebooted at any
time without notice for updates or for security reasons.
To use the other >4000 cores in OzStar you need to go via the batch system.
------------------------------------------------------------------------------
```
and you'll either be working on the `farnarkle1` or `farnarkle2` log-in node.
It doesn't matter which node you join.

You can check the groups that you are a member of using the `groups` command:
~~~
[phancock@farnarkle2 ~]$ groups
curtin ozstar_users oz983
~~~
{: .language-bash}

You probably wont be part of the `curtin` group, but you should be a member of the `oz983` group, as that is the one that we'll be using for this course.

You have access to two file systems: **home** and **fred**.
The **home** file system holds your home or `~/` directory and files, and is limited to 10GB.
It is backed up but should not be used for frequent file access (don't do work here).
The **fred** file system is a large scratch storage for general use.
It is *not* backed up, but doesn't have a purge policy, so you can store files here long term.

See [these docs](https://supercomputing.swin.edu.au/docs/1-getting_started/Filesystems.html) for more information on the file system.

> ## Create a personal work folder
> Navigate to `/fred/oz983`.
> Create a directory in here with your username
> 
> > ## Solution
> > ~~~
> > cd /fred/oz983
> > mkdir ${USER}
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

## How do I run tasks on an HPC
Currently you will have only a very minimal set of software loaded.
To see what software modules you have loaded:
~~~
module list
~~~
{: .language-bash}

~~~
Currently Loaded Modules:
  1) nvidia/.latest (H,S)   2) slurm/.latest (H,S)

  Where:
   S:  Module is Sticky, requires --force to unload or purge
   H:             Hidden Module
~~~
{: .output}

Effectively we start of with just enough tools to submit and monitor jobs.

Remember from last lesson, we need to use a scheduler in order for our jobs to be run on the compute nodes:
![JobQueuing]({{page.root}}{% link fig/queueing_infog.png %})


### Querying the queue system
Before we run any jobs we need to see what resources are available.
One way to do this is using the `sinfo` command which will report information similar to:

~~~
[phancock@farnarkle2 oz983]$ sinfo
PARTITION   AVAIL  TIMELIMIT  NODES  STATE NODELIST
skylake*       up 7-00:00:00      2   resv john[1-2]
skylake*       up 7-00:00:00    114    mix bryan[1-8],john[3-7,9-21,23-110]
skylake*       up 7-00:00:00      2  alloc john[8,22]
skylake-gpu    up 7-00:00:00      2   resv john[1-2]
skylake-gpu    up 7-00:00:00    114    mix bryan[1-8],john[3-7,9-21,23-110]
skylake-gpu    up 7-00:00:00      2  alloc john[8,22]
knl            up 7-00:00:00      4   unk* gina[1-4]
sstar          up 7-00:00:00     10 drain* sstar[107-109,122,130,140,146-147,155,164]
sstar          up 7-00:00:00      1   unk* sstar154
sstar          up 7-00:00:00      2   resv sstar[024,151]
sstar          up 7-00:00:00     45  alloc sstar[011-023,025-032,110-121,123-129,152,165-167,301]
sstar          up 7-00:00:00     31   idle sstar[101-106,131-139,141-145,148-149,153,156-163]
gstar          up 7-00:00:00      2 drain* gstar[101,204]
gstar          up 7-00:00:00     49   unk* gstar[011-059]
gstar          up 7-00:00:00      3   idle gstar[201-203]
datamover      up 1-00:00:00      4   idle data-mover[01-04]
osg            up 7-00:00:00     10   idle clarke[1-10]
~~~
{: .output}

The columns are as follows:
1. The **PARTITION** information.
A partition is a logical group of nodes that have been collected together for some common purpose.
You can think of a partition as a "queue" on a cluster.
Different partitions will have different intended uses.
See the [documentation](https://supercomputing.swin.edu.au/docs/2-ozstar/oz-partition.html#available-partitions) for a description of the different partitions.

2. The **AVAIL**ability of the given partition. Up = useable, down/drain/inact = not usable.

3. The maximum **TIMELIMIT**  that jobs can have in this queue.
Each job can request up to this amount of time.
This is not cpu time but regular time measured by a clock on the wall (a.k.a wall time).

4. The number of **NODES** in this current configuration.

6. The **STATE** of each node. 
   1. Reserved - not in use, but reserved for future use
   2. Allocated - in use will return to idle when complete
   3. Mix - Nodes which are partially allocated and partially idle
   4. Idle - not in use
   5. Unknown - ?
   6. Draining - in use but will not be idle when complete (eg going into maintenance or shutdown)

7. **NODELIST** A list of node names (shortened). Check out [this FAQ](https://supercomputing.swin.edu.au/docs/1-getting_started/FAQ.html#what-s-with-the-weird-machine-names) for what the node names mean.


The table has been grouped so that nodes in the same partition and same state will appear on a single line.
In the example above, you can see that the partition called `datamover` has 4 nodes available, and all of them are currently idle.
The `skylake` partition has a total of 118 nodes.
Note that the nodes called `bryan[1-8]` appear in both the `skylake` and `skylake-gpu` partition, while the `gina[1-4]` nodes are only available in the `knl` partition. 

The `sinfo` command tells us about the state of the different partitions.
If we want to learn about the jobs that have been submitted then we can use `squeue`.
This will show us what jobs are currently in the queue system.

~~~
> squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          29254695 datamover dd_oss.b  rhumble PD       0:00      1 (Dependency)
          29173490 datamover dd_oss.b  rhumble PD       0:00      1 (Dependency)
          29054140 datamover dd_oss.b  rhumble PD       0:00      1 (Dependency)
        29242343_0   skylake n_polych bgonchar PD       0:00      4 (Resources)
          29279155   skylake full.195     qliu PD       0:00      5 (Priority)
          29054138 skylake-g dd.batch  rhumble PD       0:00      1 (Dependency)
          29290681 skylake-g  KGAT_11      bli  R    1:36:57      1 john96
          29291743 skylake-g      zsh  tdavies  R    1:02:26      1 john8
          29291493 skylake-g Train_NF cchatter  R    1:08:59      1 bryan1
          29054139     sstar dd_oss.b  rhumble PD       0:00      1 (Dependency)
          29292308     sstar SanH2-rD mmanatun  R       8:02     31 sstar[101-106,131-139,141-145,148-149,153,156-163]
          29286615     sstar   0_full avajpeyi  R    2:54:14      1 sstar301
~~~
{: .output}

The list above shows us the jobs that are running.
Each job has a unique **JOBID**.
Jobs that are waiting to start will have a **TIME** of 0:00, and then a reason in the final column.
Jobs that are currently running will have a TIME that shows the *remaining* time for the job, and a **NODELIST** of which nodes the jobs are running on.

Running `squeue` will often give you a *very long list* of jobs, far too many to be useful.
Using `squeue -u ${USER}` will limit the list to only *your* jobs.


## Running Jobs
There are two types of jobs that can be run: either a batch job where SLURM executes a script on your behalf, or an interactive job whereby you are given direct access to a compute node and you can do things interactively.


### Interactive jobs
We will start with a simple **interactive** job.
Use `salloc` to start an interactive job.
You should see a sequence of output as follows:
~~~
[phancock@farnarkle2 oz983]$ salloc
salloc: Pending job allocation 29294281
salloc: job 29294281 queued and waiting for resources
salloc: job 29294281 has been allocated resources
salloc: Granted job allocation 29294281
salloc: Waiting for resource configuration
salloc: Nodes john16 are ready for job
[phancock@farnarkle2 oz983]$ ssh john16
[phancock@john16 ~]$ 
~~~
{: .output}

Note that salloc will create an allocation for us, but not always join us to the relevant node.
To join the node we can use ssh.
Note that my command prompt was still `@farnarkle2` (a login node), but i have to join `@john16` to access the allocated resources.

When you finish with an interactive session you can exit it using `exit` or `<ctrl>+d` to end the ssh session, and then again to exit the job allocation.

~~~
[phancock@john16 ~]$ squeue -u ${USER}
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          29294281   skylake interact phancock  R       1:20      1 john16
[phancock@john16 ~]$ exit
[phancock@farnarkle2 0z983]$ exit
salloc: Job allocation 29294281 has been revoked.
[phancock@farnarkle2 0z983]$
~~~
{: .output}

Interactive jobs are good for debugging your code, but usually leave the compute node idle whilst you read/think/code.
Interactive jobs should be used sparingly, in preference of batch jobs.

### Batch jobs
The most common type of job that you will run on an HPC is a batch job.
A batch job requires that you have a script that describes all the processing that you will need to do, as well as an estimate of the resources that you'll need.
With these bits of information the SLURM scheduler can then appropriately assign you resources and execute the work.

Lets run a basic hello world script, watch it run, and then pick apart what happened.

> ## The python script
> `/fred/oz983/KLuken_HPC_workshop/hello.py`
> ~~~
> #! /usr/bin/env python
> 
> import os
> 
> host = os.environ["HOSTNAME"]
> print(f"Hello from the world of {host}")
> ~~~
> {: .language-python}
{: .callout}

> ## The bash script
> `/fred/oz983/KLuken_HPC_workshop/first_script.sh`
> ~~~
> #! /usr/bin/env bash
> #
> #SBATCH --job-name=hello
> #SBATCH --output=res.txt
> #
> #SBATCH --ntasks=1
> #SBATCH --time=05:00
> #SBATCH --mem-per-cpu=200
> 
> # load the python module
> module load python/3.8.5
> 
> # move to the work directory
> cd /fred/oz983/${USER}
> # do the work
> python3 ../KLuken_HPC_workshop/hello.py | tee hello.txt
> sleep 120
> ~~~
> {: .language-bash}
{: .callout}

We then submit the batch script to SLURM using the `sbatch` command from our own directory, and view our job list:
~~~
[phancock@farnarkle2 phancock]$ sbatch ../KLuken_HPC_workshop/first_script.sh 
Submitted batch job 29294385
[phancock@farnarkle2 phancock]$ squeue -u ${USER}
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          29294385   skylake    hello phancock PD       0:00      1 (Priority)
~~~
{: .output}

Initially our jobs is not running because it doesn't have high enough priority, but after a minute or so we see:

~~~
[phancock@farnarkle2 phancock]$ squeue -u ${USER}
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          29294385   skylake    hello phancock  R       0:05      1 john3
~~~
{: .output}

The script will take a few seconds to run the python script, and then sleep for 2 minutes so that we have a chance to catch it in the queue.

We'll get the out put ina file called `res.txt`, which sits in the directory from which we submitted the sbtach job.

~~~
[phancock@farnarkle2 phancock]$ more res.txt 
Hello from the world of john3
~~~
{: .output}

Now let's review what we did:
1. Created a python script which did all the "hard" work.
2. Created a bash script which told SLURM about the resources that we needed, and how to invoke our program.
   1. The `##SBATCH` comments at the top of the bash script are ignored by bash, but are picked up by SLURM.
      1. `--job-name` is what shows up in the `squeue` listing. It defaults to your script name, but can be renamed here.
      2. `--output` is the location of the file that will contain all of the `STDOUT` from the running of your program
      3. `--error` (not used above) is the location of the file that will have the `STDERR` from your program
      4. `--ntasks` tells SLURM how many tasks you'll be running at once
      5. `--time` is the (wall) time that this job requires. If your job does not complete in this time it will be killed.
      6. `--mem-per-cpu` will tell SLURM how much ram is required per cpu (can also use just `--mem`) default is in MB, but you can use gb
   2. Load the software that we'll need using the `module` system
   3. Move to the directory which contains the code we are running
   4. Run the python code and copy the stdout to a file
   5. Sleep for 120 to simulate a job that takes more than a few seconds to run
3. Submitted the job to the scheduler using `sbatch`
4. Watched the progress of the job by running `squeue -u ${USER}` (a few times)
5. Inspected the ouput once the job completed by viewin `res.txt`.

## Building a workflow
Once we know how to submit jobs we start thinking about all the jobs that we could run, and quickly we get into a state where we spend all our time writing jobs, submitting them to the queue, monitoring them, and then deciding which jobs need to be done next and submitting them.
This is time consuming and it's something we should get the computers to do for us.
If we think about the work that we need to do as discrete jobs, and then join multiple jobs together to form a workflow, then we can implement this workflow on an HPC using the scheduler system.

Below is a generic workflow with just three parts.

![WorkFlow]({{ page.root}}{% link /fig/SimpleFlow.png%})

The purple boxes represent work that needs to be done, and the arrows represent dependency of tasks.
You cannot process the data until you have retrieved the data, and you have to process the data *before* you do the cleanp.

In the above diagram we would say that there is a **dependency** between the tasks, as indicated by the arrows.
Slurm allows us to create these dependency links when we schedule jobs so that we don't have to sit around waiting for something to complete before submitting the next job.
When we schedule a job with `sbatch` we can use the `-d` or `--dependency` flag to indicate these links between our jobs.

> ## Dependency example
> Try the following in your directory.
> Remember to replace the `123456` with the actual job id that is returned to you
> ~~~
> [user@host mydir]$ sbatch ../KLuken_HPC_workshop/first_script.sh
> Submitted batch job 123456
> [user@host mydir]$ sbatch -d after:123456 ../KLuken_HPC_workshop/second_script.sh
> [user@host mydir]$ squeue -u ${USER}
> ~~~
> {: .output}
> > ## Example output
> > ~~~
> > [phancock@farnarkle1 phancock]$ sbatch ../KLuken_HPC_workshop/first_script.sh
> > Submitted batch job 29442667
> > [phancock@farnarkle1 phancock]$ sbatch -d after:29442667 ../KLuken_HPC_workshop/second_script.sh
> > Submitted batch job 29442668
> > [phancock@farnarkle1 phancock]$ squeue -u ${USER}
> >              JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
> >           29442668   skylake no_hello phancock PD       0:00      1 (Dependency)
> >           29442667   skylake    hello phancock PD       0:00      1 (Priority)
> > ~~~
> >{: .output}
> {: .solution}
{: .challenge}

In the above example you'll see that the first job submitted was not running due to `(Priority)`.
This means that the job in in the queue but that it is waiting for other jobs to complete before it can be run.
The second job, however, is not running due to `(Dependency)`, and this is because it's waiting for the first job to complete.

~~~
[phancock@farnarkle1 phancock]$ sbatch --begin=now+120 ../KLuken_HPC_workshop/start.sh
Submitted batch job 29442061
[phancock@farnarkle1 phancock]$ sbatch -d afterok:29442061 ../KLuken_HPC_workshop/branch.sh
Submitted batch job 29442065
[phancock@farnarkle1 phancock]$ sbatch -d afterok:29442065 ../KLuken_HPC_workshop/collect.sh
Submitted batch job 29442066
[phancock@farnarkle1 phancock]$ watch squeue -u ${USER}
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          29442061   skylake    start phancock PD       0:00      1 (BeginTime)
          29442066   skylake  collect phancock PD       0:00      1 (Dependency)
    29442065_[1-6]   skylake     ngon phancock PD       0:00      1 (Dependency)


             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          29442061   skylake    start phancock CG       0:04      1 john31
          29442066   skylake  collect phancock PD       0:00      1 (Dependency)
    29442065_[1-6]   skylake     ngon phancock PD       0:00      1 (Dependency)


             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
        29442065_6   skylake     ngon phancock PD       0:00      1 (Priority)
        29442065_5   skylake     ngon phancock PD       0:00      1 (Priority)
        29442065_4   skylake     ngon phancock PD       0:00      1 (Priority)
        29442065_3   skylake     ngon phancock PD       0:00      1 (Priority)
        29442065_2   skylake     ngon phancock PD       0:00      1 (Priority)
          29442066   skylake  collect phancock PD       0:00      1 (Dependency)
        29442065_1   skylake     ngon phancock PD       0:00      1 (Priority)

             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          29442066   skylake  collect phancock PD       0:00      1 (Priority)

             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
~~~
{: .output}

Talk about job dependencies for making one job run after another.
Write an example where the first job makes a file, and the second job changes the file.
Submit jobs and look at them in `squeue`.

Talk about array jobs, maybe do an example as well.
Note the `%A` and `%a` notation for job logs in the slurm directives.
Show how to use the job array number to control what each of the jobs do -> use the array ID as the row number in a file which then gives the input params for our program to run.

Talk about job templates and how to use SED to fill in the template info.

- SLURM
- Slurm scripts
- Sbatch, salloc, sinfo
- Job dependencies
- Array jobs
- Flavours of nodes and different partitions/queues (revisit)
- How to create a workflow using templates (brief because Nextflow will replace this)

## How do I install software on an HPC
- Environments (make)
- Already built modules (load)

### Containers
[Singularity/Apptainer](https://apptainer.org/)/Docker
What containers are.

How to use them.

How to build them.

## Benchmarking
- Benchmarking and tracking resource usage
- Python - scalene
- Nextflow - report.html
- generic / other?
