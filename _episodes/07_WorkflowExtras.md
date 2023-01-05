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
- is there an existing workflow that you can modify
- time requirements
  - processing, time per run as well as number of expected runs
  - development, time invested to run the workflow successfully the first time
  - maintenance, additional time required for further runs fo the workflow including adapting to changes in input/output requirements or underlying software/hardware
  - cost of opportunity, will you actually save any time by automating a workflow

## Advanced container considerations
Two things to consider when building containers:
- ease of use
- ease of development

Related to the above are questions about how much software/data needs to live in each container.

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


You can create a container and define an entrypoint so that you are able to call the container as if were just an application.
This micro-service type model can work well, especially if the inputs/outputs are text based.

### How containers work together
- micro services vs one container to rule them all
- software within the container or on local system
- composing containers


