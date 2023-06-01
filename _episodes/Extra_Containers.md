---
title: "Constructing Containers"
teaching: 120
exercises: 0
questions:
- "How do I build containers?"
- "How do I plan a container build?"
- "Should I use multiple containers?"
objectives:
- "Understand that container building process"
- "Understand how containers communicate with each other"
keypoints:
- "Practice makes better"
---


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

