---
title: "Workflow Planning"
teaching: 15
exercises: 15
questions:
- "What other training is available?"
objectives:
- "Summarize and recap the lessons"
keypoints:
- "Practice makes better"
---

Article about different [workflow managers](https://www.nature.com/articles/s41598-021-99288-8).

## Workflow planning
- how to plan / structure a workflow
  - logical flow of work 
  - practical considerations
  - breaking a process into multiple smaller processes
  - merging small processes into one larger one
- how to deploy this on a cluster
- example workflow to develop

Consideration for workflow planning
- desired output 
- required input
- what artifacts will be useful for your development / debugging
- is there an existing workflow that you can modify
- time requirements
  - processing, time per run as well as number of expected runs
  - development, time invested to run the workflow successfully the first time
  - maintenance, additional time required for further runs fo the workflow including adapting to changes in input/output requirements or underlying software/hardware
  - cost of opportunity, will you actually save any time by automating a workflow
- logical requirements
  - which part of the workflow depends on other parts being complete
  - are there parts of the workflow that can be run in parallel
  - at what point can the work be check-pointed via a file/variable
- NextFlow will only submit tasks to a work scheduler when they are ready to run, so long dependency chains cannot be leveraged
  - A large number of small tasks will often not make the scheduler happy, so try for a smaller number of medium to large tasks
  - The startup / teardown time for a job in SLURM can be ~10-30s, so multiple jobs which take less than a minute to complete will waste a lot of time

Prototyping a workflow
- Start designing your workflow using a pen/paper or a drawing app like [draw.io](https://app.diagrams.net/)
- Begin at the highest most generic level
- Copy and edit your workflow to include more specifics
- Continue drilling down until you are at the point of "run this code with this file/value"
- Pay attention to any decision points that occur in your workflow
- Consider if different branches need to rejoin each other
- Start with a minimum working example and then expand from there
  - only add complexity when needed


When to update a workflow
- Break processes into smaller parts when:
  - You want to increase the checkpointing frequency
  - You want to increase the parallelism
  - Your process is consuming too many resources (time, CPU, RAM) to run efficiently
- Combine processes into one when:
  - You find yourself passing too many values/files between processes
  - The startup and tear down overhead for each process is a significant fraction of the resource usage
  - You have multiple processes that do largely the same thing (reduce technical debt)
    - Eg, make a single process which has two very similar modes of operation controlled by a flag or inferred from the input data
- 

### Example - ANNz

Workflow is to run the `annz_class.py` script with 100 different seed values and then aggregate / summarize over the 100 different runs.
Currently this is set up as a SLURM array job.

To work as a NextFlow job we would:
- write the seed values into a file, one per line
- construct an initial channel by reading from this file
- create a process that calls the `annz_class.py` script with a given seed value
  - The output of this script could be a file with stats from the given run
- create a aggregation process that will `.collect` the output from the previous script and provide a summary of the data into a single output file

The classifier process would use the existing `annz_latest.sif` image, but the aggregator process could run with a generic python container.

NextFlow advantages:
- seed values are defined in one file only (rather than multiple `.py` files)
- if jobs are interrupted or fail for some reason they can be automatically restarted
- Re-running the workflow with an extra +20 seed values, will not re-run the already completed 100 versions
- If the aggregation process is modified during development, the workflow up to this point is checkpointed and can be easily re-run without needed to wait ~36h.
