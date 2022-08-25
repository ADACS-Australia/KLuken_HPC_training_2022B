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

Use `sinfo` to see what resources are available.
Check out [this FAQ](https://supercomputing.swin.edu.au/docs/1-getting_started/FAQ.html#what-s-with-the-weird-machine-names) for what the node names mean

Use `salloc` to start an interactive job.
Load the python module with `module load python/3.8.5`
Run some basic python script.
Run  `squeue -u ${USER}` to see the status of your jobs.
exit the interactive job with ^d or `exit` and then have a nother look at the queue.
Look in your home directory for stdout and stderr logs.

Write a script that will do the same as the above but not in an interactive session.
Talk about all the SLURM directives.
Submit the job, watch it run via `squeue`, and then view the logs.

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

## Benchmarking
- Benchmarking and tracking resource usage
- Python - scalene
- Nextflow - report.html
- generic / other?
