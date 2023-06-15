---
title: "Workflow Planning"
teaching: 120
exercises: 0
questions:
- "What is the workflow for creating a workflow?"
objectives:
- "Learn about the different stages of workflow development"
- "Understand some of the constraints of NextFlow"
keypoints:
- "Planning is critical"
- "Avoid premature optimization"
---

## Background
This lesson assumes knowledge from the [NextFlow workshop](https://carpentries-incubator.github.io/Pipeline_Training_with_Nextflow/) run previously by ADACS.
In the previous lesson we focused mostly on:
- What is a workflow?
- What is NextFlow?
- How to use NextFlow to create a workflow
- How to use configurations to make a workflow portable and re-usable
- How to make a workflow user friendly

In this lesson we'll be focusing more on how you plan a workflow.
We'll be using NextFlow as our reference language, but much of what we cover here is applicable to other workflow managers.


<!-- Article about different [workflow managers](https://www.nature.com/articles/s41598-021-99288-8). -->

## Workflow planning

### 0. Desired outputs or outcomes
What is the end goal of your workflow?
At this point we are not considering how we get to our goal, but clearly defining what the goal is.
Once you have this clearly defined the rest of your planning is simply: "How do I get here".
You may need to revise your goals as you find new opportunities or discover that something isn't quite possible, but a clear goal always helps the planning process.

### 1. Required data
What data is required to meet your goal?
Some of the data that you need will be available right now, other data you may need to collect or download.
Some data may not be available until you are ready to run your workflow (e.g. Current weather conditions, or the date/time of execution).
You don't need to **have** all the data in hand right now, but you should know where/when you can obtain the data.

### 2. Logical flow of work
Once you know what your input is (the start) and your goal (the end), you can start building a workflow.
Start at a very high level of abstraction and don't worry about the pesky details right now.
Start designing your workflow using a pen/paper or a drawing app like [draw.io](https://app.diagrams.net/).


### 3. Required/available software and hardware
As well as the input data you should have some idea of what software is going to be needed and where you are likely to run the workflow.
You might need to write some code to support your work, but it is often better to spend some time investigating what is already available.
Look for software that is well supported, and ideally something that is used by your colleagues so that you have a ready supply of help when needed.
Knowing what software is needed (including licenses) may influence your choice of where you'll be running the software (or the inverse).
If the HPC that you'll be using doesn't support your first choice of software then you will have to consider either alternative software choices or alternative HPC resources.
Keep in mind that you'll often use a laptop/desktop to develop and test your workflow before deploying on an HPC so you should consider the software availability on both platforms.
If you get stuck in dependency hell, then consider using containers ([intro](https://carpentries-incubator.github.io/Pipeline_Training_with_Nextflow/05-Containers/index.html), [further discussion]({{page.root}}{% link _episodes/Extra_Containers.md %})) to make your software stack more portable.


### 4. Iteratively refine your workflow
Copy your first workflow and start to drill down on the details of each of the tasks.
Break the tasks into sub-tasks.
Continue drilling down until you are at the point of "run this code with this input to make this output".


### 5. Create a prototype workflow with test data
Implement your workflow in NextFlow, and test it with a realistic, but fast to compute data set.
You may need to spend some time finding or creating such a data set, but it will be worth the effort, since you'll be using it a lot during the development and testing of your workflow.
At this point we are aiming for a workflow which **works**.
We will think about efficiency, portability, and configuration at a later point, but we need a place to start and this is what the prototype workflow is all about.

### 6. Refine and optimize your workflow
You will likely now have a workflow with a large number of tasks, but it works!
No doubt your workflow broke a lot of times before you got it to an eventual working state.
In this development process you would have been looking at the intermediate results, the scripts that were generated, and the output logs of your workflow.
These are all pieces of information that are not essential to the end goal of your workflow (you can delete them when the workflow completes), but they are very useful for debugging and development purposes.
It is good practice to intentionally include debugging and logging *artifacts* within your workflow so that you can understand how the workflow is running.

> ## Use -resume
> When you are developing a workflow it can be very useful to use the `-resume` option for NextFlow.
> This will cause NextFlow to not redo work that is already complete.
> NextFlow uses a cache/hash to figure out if the input/script for a given process has changed, and will re-use previous outputs if possible.
> Just remember that the syntax is `-resume` and not `--resume`.
>
{: .callout}

NextFlow will run each of your tasks within a separate work directory which will also include the generated run script for your task, and the error and output.
These files are hidden, so look for the following files:
- `.command.err`
  - STDERR
- `.command.out`
  - STDOUT
- `.command.log`
  - STDOUT+STDERR
- `.command.sh`
  - The command which was run by NextFlow for your process
- `.exitcode`
  - The exit code for your script (0 = ok, !0 = !ok)

You should therefore consider making your scripts emit useful information as they are running (`set +x` and `echo` are good for this), so that when things to bad you can easily figure out where and why.

Additional items that can be useful for tracking the execution of a workflow include diagnostic plots and summary statistics.

NextFlow is also able to give you a detailed summary of the execution of the workflow if you enable the following outputs:
- report
  - Generates an HTML report [example]({{page.root}}{%link files/report.html%})
- timeline
  - Creates a timeline of when each process was executed [example]({{page.root}}{%link files/timeline.html%})
- dag
  - Creates a directed acyclic graph of your workflow [example]({{page.root}}{%link files/dag.html%})

These outputs can be extremely useful in the analysis of the workflow.

### 7. Workflow development
Now that you have a workflow that works and is giving you useful meta-data about the running processes it's time to add additional features, deploy to another system, and look at optimization.

To deploy on another system you'll need to do some or all of the following:
- copy your nextflow `.config/.nf` files
- run the above and pray it works
- install software or load modules
- add a profile to your `.config` file to include information about the scheduling system (SLURM/PBS)
- build a container for your software and copy it onto your system of choice
- update your `.config` to tell NextFlow which container(s) to use

See [Nextflow Orchestration](https://carpentries-incubator.github.io/Pipeline_Training_with_Nextflow/06-Nextflow_Orchestration/index.html) for a lesson on how to setup your `.config` file to use profiles, set up process labels, and use containers.

#### Workflow optimization

The first rule of optimization is to do nothing until you have a problem.

The second rule of optimization is to work on the things that take the longest time first.

When running tasks using the local executor of NextFlow the overhead created by NextFlow setup/finishing is minimal (seconds at most), and can be ignored, unless your tasks are only a few seconds each.
If you are running using the SLURM or PBS executor then the overhead for a single task consists of:
- NextFlow setup/finishing (few seconds)
- SLURM/PBS node setup/completion (~10-30 seconds)
- Queue time (seconds - minutes - hours?)

Depending on which of the above is having the biggest impact there are different solutions:
1. NextFlow setup/finishing
  - In this case you have very very short jobs which could easily run on the local executor (head node) without causing disruption to other users
  - If you have a **lot** of these processes then consider packing jobs as per below
2. SLURM/PBS setup/completion time
  - In this case you are likely running many jobs which require only a short time to run
  - Consider packing jobs together such that your process takes an array of inputs and then work on 5/10/100 at a time
  - Depending on the resource requirements you can either run these in parallel or serial within your process using `xargs` for example
  - Ideally you would like to reduce the total amount of overheads, so some calculation and experimentation is going to be needed here
3. Queue time
  - In this case your jobs are just sitting about doing nothing for a long time before starting
  - One way to reduce the total amount of time waiting in the queue is to have fewer jobs to submit, so packing jobs as per the above could help
  - Schedulers typically find it easier to schedule shorter jobs or jobs that require fewer resources so consider reducing the wall time, cpu, RAM requested

Let's review an [example report]({{page.root}}{%link files/Beamforming.html%}) to see which tasks fit into different categories above.

Finally, once you have decided that there is not much more down time in your NextFlow workflow, you should start looking at how to optimize the code within each of the processes.
Optimization of general codes is beyond this workshop, however [this workshop](https://adacs-australia.github.io/2023-03-20-Coding-Best-Practices-Workshop/) has lessons on testing, benchmarking, profiling, optimization, and parallel computing (in Python).