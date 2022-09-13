---
title: "Accessing HPC resources"
teaching: 15
exercises: 15
questions:
- "What were those HPC resources again?"
- "How/when do I apply for time?"
- "How do I collect the information needed for my application?"
objectives:
- "Review the HPC facilities in Australia"
- "Understand how/when to apply for time (and who is eligible)"
- "Know how to benchmark a workflow with SLURM"
keypoints:
- "HPC time is like telescope time - you need science + resource justification"
- "The best way of estimating resources is to run your workflow on test data"
---

## What HPC resources are available to me?
As an astronomy researcher in Australia there are a number of systems that you can have access to as noted in the table below.

| System                                        | Kind            | Access                    |
| --------------------------------------------- | --------------- | ------------------------- |
| [Pawsey](https://pawsey.org.au/)              | HPC cluster     | NCMAS, Partner            |
| [NCI](https://nci.org.au/)                    | HPC cluster     | NCMAS, ASTAC              |
| [OzSTAR](https://supercomputing.swin.edu.au/) | HPC cluster     | ASTAC, open               |
| Local cluster                                 | ?               | People at your institute  |
| Azure / GCloud / AWS                          | Cloud computing | Anyone with a credit card |


### NCMAS
Pawsey and NCI have a linked call for proposals via the Nation (Computing) Merit Allocation Scheme (NCMAS).
Users can apply for time via the [application portal](https://my.nci.org.au/mancini/login?next=/mancini/ncmas/2023/my-application).
The call for proposals happens annually, and for this year the key dates are:

| Dates                     | Milestone                                       |
| ------------------------- | ----------------------------------------------- |
| 15 August 2022            | Applications open                               |
| 3 October 2022            | Applications close at 7:00pm AEST (5:00pm AWST) |
| 30 Nov, 1-2 December 2022 | Allocation Committee meeting                    |
| 12 December 2022          | Allocations announced                           |
| 1 Jan 2023                | Access to allocations commences                 |

Both centers provide training and information sessions related to the NCMAS scheme, with more information available [here](https://my.nci.org.au/mancini/ncmas/2023/).
To be eligible for NCMAS time the Chief Investigator must be employed at an Australian university or research institution.

### ASTAC
Australia Astronomy Limited (AAL) runs an Astronomy Supercomputer Time Allocation Committee (ASTAC) which allocates available time on NCI and OzSTAR.
The ASTAC is managed by ADACS.
ASTAC runs twice annual calls for proposals with details available via [this page](https://astronomyaustralia.org.au/committees-astronomy-australia-ltd/astac.html).
The two semesters run Jan-Jun and Jul-Dec.


### Other
In addition to NCMAS, Pawsey offer a [Partner Merit Allocation Scheme](https://support.pawsey.org.au/documentation/display/US/The+Pawsey+Partner+Merit+Allocation+Scheme), which is very similar to NCMAS in terms of application details and resourcing, however the Chief Investigator must be employed at a Pawsey Partner institution (CSIRO, Curtin University, Edith Cowan University, Murdoch University and the University of Western Australia).

OzSTAR also has a non-merit allocation scheme (which we are currently using for this workshop), whereby Australian astronomers can have access to the facility and compete for un-used resources via the SLURM scheduler.

[Azure](https://azure.microsoft.com/en-au/) / [GCloud](https://cloud.google.com/) / [AWS](https://aws.amazon.com/) all offer cloud computing services which can provide expandable computing services on a user pays basis.
Cloud computing is often not an affordable replacement for a dedicated HPC facility, however it can still be cheaper than buying and managing a system of your own, especially when you don't require the system to be run for more than a few weeks or months.

## How to apply for time
For NCMAS, ASTAC, and Pawsey Partner Scheme the the time allocation committee (TAC) are going to be looking at the following criteria when awarding time:
1. Is the proposed science impactful?
2. Does the science team have the right people and support in place to make use of the resources requested?
3. Is the resource request well justified?
4. Is the resource request proportionate to expected science impacts?

The science impact is assessed in the same way as any other grant/telescope proposal and should be approached in the same way.

The composition of the science team is important because the TAC need to have confidence that the resources will be used in an appropriate way.
Make sure that you list only the people that will be providing either a meaning scientific or computational input into the project.
Adding "big names" to a project doesn't benefit you unless there is a clear role for them to play.
Having PhD students on a project doing a large fraction of the work is not a bad thing, so long as the students are well enough supported and have an appropriate history of working with the code base or facility in question.

The thing that many people get wrong, or don't put enough effort into, is the resource request.
HPC resources are typically assigned as "kilo-service units" (kSU), with a service unit usually corresponding to 1 cpu for 1 hour of wall time.
Given that individual nodes of an HPC facility can easily have 32 cores, a 1 hour job on a single node will consume 32SU.
Some nodes are charged at a higher rate (4x for high-memory jobs, or 8x for jobs on a debug queue).
The call for proposals will outline how the service units are calculated and will require that you make an estimate of how much time you will use.
The best way to estimate how many kSU your project will need is to do some benchmarking of your code, ideally on the target system.

To avoid the catch-22 of needing to use the HPC facility to run your code in order to estimate time for the application that will grant you access to said facility, most facilities will have a small fraction of their time set aside for directors time.
Directors time is granted by the director of the facility and typically you will do this via an email to indicate that you want to use the facility to do some benchmarking work ahead of a call for proposals.
**Note** that Pawsey and NCI will not allow people to apply for directors time close to the closing date of the NCMAS so you should plan your work well in advance.

## Benchmarking
Benchmarking is the process of running code on a target system to determine the typical behavior or resource usage.
Benchmarking is different from profiling, in that with profiling we want a detailed report of what our software is doing at various times with an eye to improving the program, where as benchmarking is only interested in estimating how much resources are required to run a program in it's current state.
In the context of this workshop we are mostly interested in determining the resource usage in terms of:

1. run time
2. peak RAM use
3. CPU utilization

The peak RAM use and CPU usage will determine how many copies of our task we can run on a node at once, which we can then multiply by the total run time to estimate our kSU requirement.

Whilst it's possible to estimate the cpu/time/ram requirements by running tasks on a desktop and then "scaling up" the results, this is an unreliable method, and usually requires a buffer of uncertainty.
The best method is to run some test jobs on the target machine and then ask SLURM how much resources were used for those jobs.
The key to this method is the `sacct` (SLURM accounting) task.

In the example below I run `sacct` on a job that has completed:
~~~
sacct -j 29780362
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
29780362           test    skylake      oz983          4    TIMEOUT      0:0 
29780362.ba+      batch                 oz983          4  CANCELLED     0:15 
29780362.ex+     extern                 oz983          4  COMPLETED      0:0
~~~
{: .output}

There area few things to unpack here so lets go in order of columns:
- JobID - This shows all the jobs you asked to see. Note that there are three job steps shown here.
  - 29780362 - This is the parent job, this row will show summary attributes that include all other steps.
  - 29780362.ba+ - this is the "batch" job, what was executed within your bash script.
  - 29780362.ex+ - this is the "external" tasks that were run, typically this will be small/none in terms of resource usage
  - 29780362.0 - [Not shown above] steps that end in a `.<number>` are created each time you use `srun` to launch a task. Unless you are using mpi jobs this is not required so you may not see this.
- JobName - The name of the job/step
- Partition - The cluster name or partition that the job ran on
- Account - The account that will be charged for the resources used
- AllocCPUS - The number of CPUs that were allocated to the job
- State - The final state of the job
  - CANCELLED Job was cancelled by the user or a sysadmin
  - COMPLETED Job finished normally, with exit code 0
  - FAILED Job finished abnormally, with a non-zero exit code
  - OUT_OF_MEMORY Job was killed for using too much memory
  - TIMEOUT	Job was killed for exceeding its time limit
- ExitCode - The (highest) exit code for the job along with the signal that caused it to exit in the format exitcode:signal

In the above example, I submitted a task that requested minute wall time.
The job ran over time and was therefore cancelled by SLURM.
The SLURM controller sent [signal](https://www.computerhope.com/unix/signals.htm) 15 (SIGTERM) to the script which caused it to exit with code 0.

> ## What else can `sacct` do?
> Read the `man` pages for `sacct` and see what other reporting options are available.
> For a short hand view try `sacct -e`.
> > ## `sacct -e`
> > ~~~
> > Account             AdminComment        AllocCPUS           AllocNodes         
> > AllocTRES           AssocID             AveCPU              AveCPUFreq         
> > AveDiskRead         AveDiskWrite        AvePages            AveRSS             
> > AveVMSize           BlockID             Cluster             Comment            
> > Constraints         Container           ConsumedEnergy      ConsumedEnergyRaw  
> > CPUTime             CPUTimeRAW          DBIndex             DerivedExitCode    
> > Elapsed             ElapsedRaw          Eligible            End                
> > ExitCode            Flags               GID                 Group              
> > JobID               JobIDRaw            JobName             Layout             
> > MaxDiskRead         MaxDiskReadNode     MaxDiskReadTask     MaxDiskWrite       
> > MaxDiskWriteNode    MaxDiskWriteTask    MaxPages            MaxPagesNode       
> > MaxPagesTask        MaxRSS              MaxRSSNode          MaxRSSTask         
> > MaxVMSize           MaxVMSizeNode       MaxVMSizeTask       McsLabel           
> > MinCPU              MinCPUNode          MinCPUTask          NCPUS              
> > NNodes              NodeList            NTasks              Priority           
> > Partition           QOS                 QOSRAW              Reason             
> > ReqCPUFreq          ReqCPUFreqMin       ReqCPUFreqMax       ReqCPUFreqGov      
> > ReqCPUS             ReqMem              ReqNodes            ReqTRES            
> > Reservation         ReservationId       Reserved            ResvCPU            
> > ResvCPURAW          Start               State               Submit             
> > SubmitLine          Suspended           SystemCPU           SystemComment      
> > Timelimit           TimelimitRaw        TotalCPU            TRESUsageInAve     
> > TRESUsageInMax      TRESUsageInMaxNode  TRESUsageInMaxTask  TRESUsageInMin     
> > TRESUsageInMinNode  TRESUsageInMinTask  TRESUsageInTot      TRESUsageOutAve    
> > TRESUsageOutMax     TRESUsageOutMaxNode TRESUsageOutMaxTask TRESUsageOutMin    
> > TRESUsageOutMinNode TRESUsageOutMinTask TRESUsageOutTot     UID                
> > User                UserCPU             WCKey               WCKeyID            
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

For our current needs the relevant fields are as follows:

| Field     | Description                                                  |
| --------- | ------------------------------------------------------------ |
| TimeLimit | How much time was *allocated* to the job                     |
| Elapsed   | How much time was *used* by the job                          |
| NCPUS     | *allocated* number of CPUS                                   |
| UserCPU   | Time spent on user time (the program you ran)                |
| SystemCPU | Time spent on system time (libraries called by your program) |
| TotalCPU  | Total time spent (User + System)                             |
| CPUTime   | NCPUS * Elapsed                                              |
| ReqMem    | Requested memory                                             |
| MaxRSS    | Maximum RSS (used memory)                                    |
| MaxVMSize | Maximum VMSize (addressable memory )                         |

We can use these fields to get the following information:
~~~
sacct -j  29780362 -o JobID,TimeLimit,Elapsed,NCPUS,UserCPU,SystemCPU,TotalCPU,CPUTime,ReqMem,MaxRSS,MaxVMSize
JobID         Timelimit    Elapsed      NCPUS    UserCPU  SystemCPU   TotalCPU    CPUTime     ReqMem     MaxRSS  MaxVMSize 
------------ ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- 
29780362       00:01:00   00:01:16          4  00:03.649  00:02.291  00:05.941   00:05:04         4G                       
29780362.ba+              00:01:19          4  00:03.649  00:02.290  00:05.939   00:05:16                  572K    211236K 
29780362.ex+              00:01:16          4   00:00:00  00:00.001  00:00.001   00:05:04                   88K      4380K 
~~~
{: .output}

For this particular job we requested 4 CPUs and used 3.649 seconds of user time, 2.291 seconds of system time, for a total of 5.941 seconds, and ran for 1minute 16seconds.
The amount of time that could have been used if we had used all 4 CPU cores at 100% is 5:04 minutes, meaning that we used less than 1% of the allocated resources.
We requested 4GB of RAM but had a peak VMSize of just 212MB, meaning that we could have requested less RAM.
For my example task I would have been charged 1minute x 4 cores worth of resources, but have made use of less than 1% of those resources.

When benchmarking your jobs, it is clearly important to have the jobs run successfully.
Once a job runs successfully you can then use `sacct` to figure out the time, CPU, and RAM usage.
Note that the `sacct` system only polls the jobs at some interval (30seconds?) and therefore it is possible that the MaxVMSize will not capture short duration peaks in RAM usage.
This polling interval also means that your jobs that run overtime will not always be cancelled exactly at the wall time requested (see the example above).

For longer running jobs you can monitor their resource usage in near-real time thanks to [this nifty website](https://supercomputing.swin.edu.au/monitor/).
Take a minute to explore the site and see what kinds of efficiency people are getting in their jobs.