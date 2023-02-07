---
title: "Managing Environments"
teaching: 15
exercises: 15
questions:
- "What other training is available?"
objectives:
- "Give feedback on training"
keypoints:
- "Practice makes better"
---

## environment management
Whilst containers are a great way for running your code in a consistent environment, they are not a good way to manage your software environments for your every-day work.
There are a number of places where your work environment is set, depending on the program that you are using.

### Bash environments
When you log into a linux based system you'll have your (bash) shell settings loaded from a file called `.bashrc`.
Even if you haven't edited this file it will have a lot of default settings.
One common thing that people want to do is have some aliases set, so that they can type, for example, `ll` and have it act like `ls -lah`.
As noted toward the end of the `.bashrc` file it is recommended that you place the alias commands into a file called `.bash_aliases` which will be automatically loaded (if found) by your `.bashrc file`.

For examples I have the following defined in my `.bash_aliases` file:

~~~
alias emacs='emacsclient -t -a='
alias topcat='java -jar ~/Software/topcat/topcat-full.jar'
alias stilts='java -jar ~/Software/topcat/topcat-full.jar -stilts'
alias ds9='~/Software/ds9/ds9 -scalelims -0.2 1 -tile -cmap cubehelix0 -lock frame wcs -lock scale yes -lock colorbar yes -lock crosshair wcs'
alias py3='source ~/.py3/bin/activate'
~~~
{: .language-bash}

The first line invokes emacs in a server mode, which causes it to start faster, and for multiple invocations of emacs to be aware of each other.
The `topcat` and `stilts` aliases allow me to run these java programs without having to remember how java works.
The `ds9` command loads ds9 while choosing a bunch of default display options.
Finally the `py3` alias loads my python3 evnironment, which we'll discuss in the next section.


### Python virtual environments
It is good practice to use a python virtual environment for running your python codes.
Particularly for anyone who is developing code, having a different environment for each of the projects / apps that you are working on, can help reduce [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell).

There are two ways of creating a python virtual environment and it depends on how you install your modules.
Commonly you will use either `pip` or Anaconda (or miniconda).
Reguardless of how you create/manage your environments, running python code is the same in each and the same as if you were not using a virtual environment.

A practical approach to python environments would be to have:
- a generic environment for your day-to day messing about that will have modules installed/removed/updated whenever you want to try new things
- an environment for each project that you are working on
- a well maintained environment for any software that you are developing that will be shared with others

The last note above can be achieved by indicating the names and versions of all the modules that are needed for your software to work.
Often this `requirements.txt` (pip) or `environment.yaml` (conda) file can be created from an existing evnironment, so you can build an environment, install the modules needed untill your program works, and then save the names/versions into the required file.

### PIP - venv
To create a new environment you run `python3 -m venv --prompt PROMPT ENV_DIR`.
This will create a new directory called `ENV_DIR` and create within it all the files and executables required to run python.

To activate the environment you would then need to run `source ENV_DIR/bin/activate` (tip: create an alias for this - see above).
When activated, you will see that your command prompt changes to have `(ENV_DIR)` at the start to remind you which environment you are using.
You can only have one environment activated at a time.
Once an environment is activated any calls to python, including scripts with `#! /usr/bin/env python` in the header, will use the versrion of the interpreter and modules from the virtual environment.
To install new modules just use `pip install <module name>`.
To view the modules (including versions) installed in the current environment you can use `pip freeze > requirements.txt`.

To deactivate the environment simply by typing `deactivate`, your command prompt will return to normal.

Example:

~~~
paulhancock@GammaCrucis:~$ python3 -m venv --prompt py3 ~/.py3
paulhancock@GammaCrucis:~$ source ~/.py3/bin/activate
(py3) paulhancock@GammaCrucis:~$ pip install -r requirements.txt
... # do work, run scripts etc.
(py3) paulhancock@GammaCrucis:~$ deactivate
paulhancock@GammaCrucis:~$ 
~~~
{: .language-bash}

### Conda
If you are using Anaconda, you will have a graphic interface that you can work with to create/select/launch different environments.
See [here](https://docs.anaconda.com/navigator/tutorials/manage-packages/) for instructions on how to do that.

If you are using conda or Anaconda from the command line then the relevant instructions are:

Create a new environment with `conda create --name myenv --prefix ./envs`.
In contrast to pip, the `--prefix` specifies the location that the files will be stored, and `--name` specifies the name of the environment which will be prepended to your command line when activated.
You can optionally also include `python=3.9` to specify the version of python that you want to use in this environement (something that pip/venv doesn't easily support).

Activating a conda environment is done via `conda activate ./env` where `./env` is the directory that stores your environment.
Once activated you can install modules using `conda install <module name>`.

You can list the modules installed in the current environment using `conda env list`.
To build an `environment.yaml` file you can use `conda env export > environment.yaml`, which can then be loaded via `conda create -f environment.yaml`

## Accessing programs on a remote system
The method of choice for connecting to a remote system is `ssh`.
By default you'll have a user/pass access to a remote system, requiring to you remember and enter your password every time you access that system.
You can set up a passwordless access using ssh keys which by default  are stored in your `~/.ssh/` directory.
[This site](https://www.freecodecamp.org/news/the-ultimate-guide-to-ssh-setting-up-ssh-keys/) has a fairly straight forward description of how to set up and use ssh keys for easy remote access, including accessing GitHub.
There will eventually be some programs that you cannot use via the command line alone and so you'll have to look into options for rendering graphics from a remote system.

### Graphic programs on an HPC
When you connect to a remote system using `ssh` use either the `-X` or `-Y` flag (by default they do the same thing), and this will enable X11 forwarding.
This means that so long as you have an X11 client on your local machine (linux/Mac does, Windows see [here](https://stackoverflow.com/a/61110604/1710603)), the remote system will render graphics and then forward it to your local machine via ssh.
Note that X11 forwarding will incur a high latency, so you'll get a degraded experience compared to running on your local machine, even for 'lite' applications like emacs or vim.
Note also, that many HPC systems will either not have a local X11 server for you to connect to, or will disable the default port which is used to forward the X11 session.
If this is the case, then your friendly HPC provider will likely have a visulization node that you can use for graphic programs.
[Pawsey](https://support.pawsey.org.au/documentation/display/US/Visualisation+Documentation) provides a web-based remote desktop experience, as does [NCI](https://opus.nci.org.au/display/OOD/2.+VDI+App), while OzStar seems not to.


### Graphic programs on a remote system (which you control)
An excellent tool for connecting to a computer that you own/control without having to deal with X11 forwarding, is [NoMachine](https://www.nomachine.com/).
NoMachine (or NX) is free to install and use, and the server/client can be run on any Win/Mac/Linux system with ease.
The graphics forwarding uses a protocol different from X11, which has been optimized to provide low (-er) latency. 
Using NX, you can easily set up a service on your work desktop machine, which you can then log into via your laptop or home computer.
So as long as you have a reasonable internet connection, you do not need to duplicate your work / environment on multiple systems.