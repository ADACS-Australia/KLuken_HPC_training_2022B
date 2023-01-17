---
title: "Workflow Extras"
teaching: 15
exercises: 15
questions:
- "What other training is available?"
objectives:
- "Summarize and recap the lessons"
keypoints:
- "Practice makes better"
---

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


## Advanced container considerations
Two things to consider when building containers:
- ease of use
- ease of development

Having a single container that has all of your software and dependencies is convenient for deployment as you have a one-size-fits-all container.
However, during development, you'll often need to change your software and possibly update dependencies.
Having to rebuild your container every time you change your software can be a massive time sink.
For software that is still under active development, it is often preferable to build a container that contains all the dependencies, but keep your own code on your local machine.
When you invoke your code you can bind your working directory to some place within the container and then run your code from there.
This will drastically reduce the try/test cycle of your development.
When your code becomes stable you can then move it into the container.

Related to the above are questions about how much software/data needs to live in each container.
For the most part it is a good idea to store as little data inside a container as possible.

Options include:
- building multiple containers, one for each of the different pieces of software that you need
- building a monolithic container that contains all the software you want

A middle ground is sensible:
- group software into common requirements and use one container per group

If your workflow relies on a bunch of python code and your various scripts require a common or overlapping set of modules then build one "python" container which contains all of these requirements.
This will likely mean that you have one ~1-2GB python container which can be used for all of your scripts.

Example setup for a project:
- a container for java based applications
- a container for python based applications
- a container for 3rd party C/C++ applications which have some complicated build requirements
- a container for tensorflow which can use GPUs

If you have multiple pieces of software that need to communicate with each other then running them in separate containers will add an orchestration overhead that you probably don't want.
NextFlow can easily run a process within a given container, but specifying multiple containers for a single process isn't possible.
To get around this you'll either need to create a single container that contains everything you need for a single process, or break the process into multiple parts.

You can create a container and define an entrypoint so that you are able to call the container as if were just an application.
This micro-service type model can work well, especially if the inputs/outputs are text based.

### How containers work together
Containers make it very easy yo create a set of independent but related services that communicate with each other via an api.
This approach is called micro-services and is common for projects that are creating a web application.

A web app would have containers such as:
- A [database](https://www.postgresql.org/) to store data
- An environment for running custom code (e.g. some machine learning models)
- An [ORM](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) such as DJango that connects the database and the custom code and presents it to the user
- A web service such as [nginx](https://nginx.org/en/) that passes user requests to DJango and serves pages to the user

Getting the containers to talk to each other is then managed by specifying which containers are connected to which, and what ports they are using to communicate.
[Docker-compose](https://docs.docker.com/get-started/08_using_compose/) is a common solution that allows you to specify these connections in a configuration file.
In the above example all the containers would be running persistently waiting for user input.
Such a configuration is often not required for scientific workflows.

