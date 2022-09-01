---
title: "Parallel Computing"
teaching: 15
exercises: 15
questions:
- "What will we be learning?"
objectives:
- "Review schedule"
keypoints:
- "This is not a lecture"
---

## Types of parallelism	
One of the biggest advantages of using an HPC is that you will have access to a large number of processing cores.
These cores are often not that much faster than what you have on a desktop machine (they may even have a lower clock speed).
A desktop machine will probably have between 4-12 CPU cores, where as the individual nodes of an HPC facility will have 24-48 cores, and of course there can be hundreds or thousands of such nodes within the facility.
The key to making best use of HPC resources is thus organizing your work to make the best use of these multiple cores, and this means that you need to understand how to work in parallel.

There are many different levels of parallelism that you can work with, which we'll explore now.

### Task based parallelism
In task based parallelism we are working at the highest level of abstraction.
In this model we take our workflow and break it into discrete tasks and then ask which tasks are independent of each other.
These independent tasks can then be run at the same time in parallel.
You may see these tasks referred to as *embarrassingly parallel* tasks.

Depending on how the software is implemented there are two main ways to run tasks in parallel:
1. Run one task per compute node, with many nodes being use at once. This would typically mean running an array job.
2. Run one task per CPU within a single compute node, with just one node being used at a time. This would be facilitated with a single job script.

We can of course run hybrid models which combine the two above extremes.
Suppose we have 1000 tasks that need to be done, and each task can be completed using a single CPU core in 6hours.
Suppose we want to run this on [OzStar](https://supercomputing.swin.edu.au/ozstar/), such that we have 107 compute nodes, each with 36 CPU cores ([of which we can use 32](https://supercomputing.swin.edu.au/docs/2-ozstar/oz-slurm-create.html)).
We could use (1) above, but it would required 1,000 nodes x 6 hours of resources (6,000 node hours), and only use 1/36th of the available cores available on each node.
This is not a good use of resources.
We can't use (2) above because our nodes don't have 1000 cores in any single node.
Instead we could divide our 1,000 tasks into `1000/32 = ~ 32` jobs, and then within each job run one task per CPU (assuming it will fit within the memory constraints of the node).
This could be achieved by having a job array of 32 jobs, and each job would orchestrate the running of tasks numbered `(n-1)*32` to `(n-1)*32 + n`, and making sure that the last job in the array stops at the 1,000th task.
This method will result in 32 nodes x 6 hours of resources, or ~32 better than the previous option.
We refer to this type of organization as job packing.

Occasionally, the limiting resource on a compute node is not the number of CPU cores but the amount of RAM available.
If we can either limit or estimate the peak RAM usage of our tasks then we can adjust our job packing such that we use as much of the RAM as possible per node.
For example, we might have 192GB of RAM available per node, but have a task that uses up to 10GB of RAM, and so we'd only be able to pack 19 copies of this task onto each of our compute nodes.

If we instead had 10,000,000 tasks that each took 20 second to run this would still have a total run time of approx 6,000 node hours, the same requirement as the previous example.
However, SLURM takes some time to schedule a job, allocate resources, and then clean up when the job completes.
This time is not that long (maybe 30 seconds), so if your job has a relatively short run time (few minutes) then this overhead becomes a significant overhead.
In such cases we can pack our jobs such that we have 10,000 jobs running within a single job, but only 32 running at one time.
This is a second form of job packing.

In all of the above implementations the parallelism is being handled by a combination of SLURM and our batch job scripts.
We don't need to have any knowledge of (or access to) the source code of the program that is actually doing the compute.

In a [previous lesson]({{page.root}}{% link _episodes/03_WorkingOnAnHPC.md %}#parallel_workflows) we explored job arrays, so we won't revisit them here, however we will dive a little deeper into job packing.

TODO: a bunch of info graphics for the above, plus an exercise for the learners.

intro `xargs` and demonstrate it's use on a local machine or interactive session

show also that `srun` can be used to manage jobs slightly more easily than xargs (though only on a SLURM system), by using `srun --exclusive`

 
### Domain based parallelism

MPI and OpenMP

### Vectorized operations


## Memory models
- Shared memory with OpenMP
- Distributed memory with MPI

## Parallel processing
- With bash
- With Python
- Others (links to C/Fortran material)

Some content can be cribbed from [here](https://adacs.org.au/wp-content/uploads/2016/02/slurm.pdf)