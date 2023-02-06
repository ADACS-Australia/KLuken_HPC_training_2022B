---
title: "Git and Bash Extras"
teaching: 15
exercises: 15
questions:
- "What other training is available?"
objectives:
- "Give feedback on training"
keypoints:
- "Practice makes better"
---

## Git
Discuss various [git workflows](https://www.atlassian.com/git/tutorials/comparing-workflows):
- centralised
- feature branch
- gitflow
- forking

### Centralized workflow
In this workflow there is a single repository (usually the one on GitHub) is designated as the central repository.
When people want to make changes to the repo they pull the current version, make their changes and then push back to the central repo.
This style works well if you have only a few developers who do not work on similar parts of the code at the same time, so the expectation for conflicts is very low.
This method is simple to understand and easy to work with.
If you are the sole developer/user of your repository then this is probably how you will work.

Atlassian have a [deeper discussion](https://www.atlassian.com/git/tutorials/comparing-workflows) about the pro/con of working in this way.
![CentralizedWorkflow](https://wac-cdn.atlassian.com/dam/jcr:0869c664-5bc1-4bf2-bef0-12f3814b3187/01.svg?cdnVersion=714)

### Feature branching workflow
Similar to the centralized workflow except that when changes are going to be made to the repo a developer will create a branch to work on those changes.
As a feature is being developed changes will often break the functionality of the software so keeping all these changes in a branch separate from `main` will mean that there is always a 'known working' version of the code that people can use.
You could consider the local copies of a repo in the centralized workflow to have a similar purpose to the branches in the feature branching workflow.
However, a key difference is that by having the branches stored in the repository, you can have multiple people seeing and working on these branches.
Another difference is that you can make a different branch for each feature, and have multiple features being developed at the same time.

Consider the case where you are working on a new feature for your code.
You pull the main branch from the centralized workflow and start developing that feature.
As you are part way through you find a bug that needs to be fixed in the code.
You now either have to make that bug fix part of the feature development, meaning that you cant push it back to the main repository until your development is complete, or you have to discard your development in order to fix the bug, before retuning.
Now consider how this would work if you used a feature branching workflow.
You make a new branch from `main` for `feature-1` and start working on it.
You notice a bug in the main code so you create a new branch from `main` called `bugfix-1`.
You fix the bug in `bugfix-1` and then merge it back to `main` and then also to `feature-1` (possibly using a `merge rebase main`).
You can now return to developing on `feature-1` without having to backtrack.

![FeatureBranching](https://wac-cdn.atlassian.com/dam/jcr:09308632-38a3-4637-bba2-af2110629d56/07.svg?cdnVersion=745)

Another advantage to the feature branching workflow is that by having the branches exist in the central repo, you can have multiple people working on (testing/reviewing) them at the same time.
The feature branching workflow includes a new operation that isn't used in the centralized workflow: a pull (or merge) request.
A pull request (or PR) is initiated on the central repository (eg, GitHub), and is a request to pull changes from one branch into another.
The idea is that developer A will make a bunch of changes in their feature branch, and then when they are happy with the changes, they will create a PR to merge these into another branch (usually main).
Good practice is to then have a different developer act as a reviewer for these changes.
Developer B will look at what the feature branch is trying to address, what has changed, and check that tests are still passing, new tests have been created, and documentation has been created/updated.
Once the reviewer is happy they approve the PR and the feature branch is merged.
For solo developers the PR is not always required, but is still sometimes used as it can cause automated testing to be run (see CI/CD later).
Even in small teams, it can be very beneficial to require all changes to the main branch to be done via pull requests from a feature branch, with some code review and discussion before the PR is accepted.
Again, Atlassian have a [more detailed description](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) of the feature branching workflow.


### GitFlow
GitFlow takes the feature branching workflow and adds some additional structure.
The features are:
- A main branch that is changed only by pull requests from a development branch
- Tagging of the main branch with versions corresponding to updates to the branch
- A pull request onto the main branch is considered a "release"
- A persistent development branch from which all features are branched and then merged back onto
  - Feature branches are deleted once merged back into the development branch
  - Critical bugs in the main branch can be fixed and merged into both main and development branch. We call this a "hotfix" and it's considered to be messy.
- Tracking of issues is done via GitHub issues, and to work on an issue you will create a branch with that issue name or number (eg. `114-https-bugfix`)

![GitFlow](https://wac-cdn.atlassian.com/dam/jcr:34c86360-8dea-4be4-92f7-6597d4d5bfae/02%20Feature%20branches.svg?cdnVersion=714)

When an issue is resolved, a pull request is made.
Linking the issue to the pull request will cause the issue to be closed when the PR is accepted and merged.
GitHub will automatically prompt you to delete the branch once it is merged.

This workflow is very good for working in teams as it will allow you to more easily incorporate project management into your development workflow.
When working in a team, rules or guides for branch names, testing and documentation requirements, and coding style, should all be agreed on and ideally documented within the repository itself.

### Forking a repository

![ForkingWorkflow](https://wac-cdn.atlassian.com/dam/jcr:642c56e3-ddc6-43ff-ab86-c5cd845afd05/03.svg?cdnVersion=714)

Note that:
- forking = cloning
- sync a project to two repositories
  - `git remote add alt <url>`
  - `git fetch alt`
  - `git push alt/branch`


### CI/CD
- writing tests with pytest
- running tests locally
- running tests on github
- auto documentation with sphinx
  - docstrings -> api
  - .md / .rst -> html
- auto-doc on github
  - readthedocs.io on merge into main


## Bash (and other unix shells)
Your favorite language may be python, but that doesn't mean that every problem should be solved using python.
More generally, it is good practice to know the basics of many different languages/tools, so that when you want to solve a problem or complete some task, you have a range of tools at your disposal, and can choose the one that is best suited for the job.
A tool that everyone has access to is the terminal (usually a Bash shell), and there are a number of powerful tools that come along with it that we will explore here.

### grep/sed/awk and regular expressions
These tools are extremely useful for manipulating streams (or files) of text.
`grep` (or egrep) is a filtering program that will return lines of text that match a given pattern.
`sed` is the stream editor, which is designed to match *and replace* text according to a pattern that you pass.
Finally there is `awk` which is a programming language geared toward text editing.
In order to ge the most out of these tools there is a meta-skill that is needed and that is regular expressions (or "regex"), which is a language for describing patterns.

Unknowingly you have already been using regular expressions when you execute a command such as:
```
ls *.fits
mv output?.dat output/.
```
{: .language-bash}

These "wildcards" of `*` and `?` in the above example are shortcuts for "anything" (`*`) or "just one character (`?`).
We can extend this further by using `[0-9]` which means "one of the digits between 0-9", or `[a-z]` to indicate "any lower case letter".
For example `ls 202[012]` would list all files/directories with a name that is either 2020, 2021, or 2022.
Similarly we could do `ls 201?` to show files/directories such as 2010, 2011, ..., 2019, but also 201a, 201X, and even 201#.

### Regular expression pattern matching

| Pattern            | Matches                                                              |
| ------------------ | -------------------------------------------------------------------- |
| `^`                | start of a line                                                      |
| `$`                | end of a line                                                        |
| a single character | that one character                                                   |
| `*`                | match zero or more of the preceding symbol                           |
| `+`                | match one or more of the preceding symbol                            |
| `?`                | match zero or one of the preceding symbol                            |
| `[]`               | one of the characters within the braces                              |
| `[^]`              | any character that is **not** with the braces                        |
| `()`               | nothing. Creates a group from whatever is matched within the braces. |
| `\n`               | the group number n (1-9)                                             |
| `\w`               | any word character (alphanumeric char), `\W` inverts                 |
| `\s`               | any white space char (eg, tab/space), `\S` inverts                   |
| `\t`               | the tab character                                                    |
| `[[:lower:]]`      | any lower case char, same as `[a-z]`                                 |
| `[[:upper:]]`      | any upper case char, same as `[A-Z]`                                 |
| `[[:alpha:]]`      | any alphabet character same as `[a-zA-z]`                            |
| `[[:digit:]]`      | any numerical digit (same as `[0-9]`)                                |
| `[[:alnum:]]`      | any alphanumeric character (same as `\w`)                            |
| `\meta`            | the meta-character itself                                            |

A more complete summary of the various matching options is given on this [cheat sheet](https://cheatography.com/davechild/cheat-sheets/regular-expressions/).
Regular expressions can get extremely complicated and suffer from the curse of being an essentially a read only language.
A deep dive on regular expressions isn't worth doing, however a basic understanding is extremely helpful as it will allow you to do some fancy things with `grep` and `sed`.

Examples:
- `grep '^#SBATCH' jobfile.sh` will match lines that **start with** `#SBATCH`
- `grep 'amen\.$' bible.txt` will match lines that **end with** "amen." (including the trailing period)
- `grep '[Hh]ello' myfile.txt` will match instances of **either** "Hello" or "hello" within the file
- `grep '0[0-9]' ids.txt` will match pairs of digits from 00 -> 09
- `grep 'a(b|c)\1' text.dat` will match abb or acc but not abc. The \1 represents a copy of the group captured within the braces `()`.
- `grep '\*' -` will match the character `*`

Repeatedly testing regex patterns can be a pain, but I find [regex101.com](https://regex101.com) to be a useful place to test my regex on some example text.
The big strength of this site is that it shows you *how* the matches are being made and what each element of your pattern is matching to.
The site uses standard regex so you can't simply copy/paste into your sed/grep commands since you'll have to escape some characters (like `(` or `{`)

This site [GNU sed REPL](https://sed.js.org/) allows you to test out your `sed` commands either on example input, or input that you post yourself.
There is no highlighting being done, but you can directly copy the regex you use here onto your command line and it will work.

### back to grep/sed/awk
`grep` (or `egrep`) is a very handy tool that will allow you to quickly scan through files and find lines that match a pattern.

For example, if we have a very long log file, we can search for lines that contain the word "WARN" using: `grep 'WARN' logfile.txt`.
We could also search through a bunch of python scripts to find lines that make calls to the print function by using: `grep 'print' *.py`.

With some regex help we can look for lines that have been commented out in our scripts by searching for a # at the start of the line: `grep '^#' *.py *.sh *.c`.
If we want to exclude the "#SBATCH" and "#include" directives we can require that the line starts with a # followed by a space: `grep '^# ' *.py *.sh *.c`.
Furthermore we can allow the line to start with some indentation of space/tab combo: `grep '^\s*# ' *.py *.sh *.c`.
Note that this last example will not include lines that start with code, and have a trailing comment.

By default `grep` will just print the lines that match, and if you are searching multiple files, it will prepend the line with `<filename>:`.
Here are some commonly used parameters that you can use to modify the behavior of `grep`:

| option | effect                                                            |
| ------ | ----------------------------------------------------------------- |
| -i     | ignore case when making matches                                   |
| -v     | print **non**-matching lines                                      |
| -c     | just count the number of matching lines                           |
| -l     | just print the names of files which have at least 1 matching line |
| -n     | prepend the line number to the output                             |
| -A n   | also print n lines after a matching line, default 0               |
| -B n   | also print n lines before a matching line, default 0              |
| -C n   | do -A n and -B n at the same time, default 0                      |
| -r     | recurse into directories                                          |


`sed` is designed to let you edit a text file or stream, and requires that you provide a pattern for matching, and then a replacement for the text that is matched.
`sed` is particularly useful for editing files in bulk.
When developing this course I had some example job scripts that had lines that looked like the following:
```
#! /usr/bin/env bash
#
#SBATCH --job-name=start
#SBATCH --output=/fred/oz983/phancock/start_%A_out.txt
#SBATCH --error=/fred/oz983/phancock/start_%A_err.txt
#
#SBATCH --ntasks=1
#SBATCH --time=00:05
#SBATCH --mem-per-cpu=1G
```
{: .language-bash}

These scripts worked for me, however they will not work as inteded for others because I have hardcoded the output/error directories to **my** work directory.
What I want to do is replace instances of `/phancock/` with `/%u/` so that SLURM will replace the `%u` with the username of whoever is running the script.
To achieve this I firstly need to find all instances of the pattern `/phancock/`, and for this I use grep:
```
grep '^#SBATCH.*/phancock/' *.sh
```
{: .language-bash}

I have included the `^#SBATCH` so that I only match lines that are in the SLURM directives header section, and then use `.*` to match any set of characters between the `#SBATCH` section and the `/phancock/` section.
If I were less pedantic and simply matched to `phancock` then I might end up matching lines elsewhere in the file.

```
branch.sh:#SBATCH --output=/fred/oz983/phancock/ngon_%A-%a_out.txt
branch.sh:#SBATCH --error=/fred/oz983/phancock/ngon_%A-%a_err.txt
collect.sh:#SBATCH --output=/fred/oz983/phancock/collect_%A_out.txt
collect.sh:#SBATCH --error=/fred/oz983/phancock/collect_%A_err.txt
mpi.sh:#SBATCH --output=/fred/oz983/phancock/MPI_Hello_%A_out.txt
mpi.sh:#SBATCH --error=/fred/oz983/phancock/MPI_Hello_%A_err.txt
openmp.sh:#SBATCH --output=/fred/oz983/phancock/OpenMP_Hello_%A_out.txt
openmp.sh:#SBATCH --error=/fred/oz983/phancock/OpenMP_Hello_%A_err.txt
start.sh:#SBATCH --output=/fred/oz983/phancock/start_%A_out.txt
start.sh:#SBATCH --error=/fred/oz983/phancock/start_%A_err.txt
```
{: .output}

We can see that there are 5 files, each with 2 instances of the mistake that I want to correct.

To replace all instances of `/phancock/` with `/%u/` I can use `sed` with the following syntax:
```
sed -e 's[separator][match pattern][separator][replacement text][separator]g' <filename>
```
{: .language-bash}

Usually people will use `/` or `:` as the separator, depending on what characters are being used in the match/replacement string. 
Any character can be used.
The `s` at the start of the string stands for *substitute* and the trailing `g` means *globally* (eg, all lines, possibly multiple times per line).
We can use the match pattern from our `grep` command above, however we don't want to replace the entire line, just the `/phancock/` section.
Thus we need to create some *capture groups* using `\( \)`.
Our `sed` command would look like this:

```
sed -e 's:\(^#SBATCH.*\)/phancock/:\1/%u/:g' branch.sh
```
{: .language-bash}

We have captured the first part of the line using `\(^#SBATCH.*\)` into group 1.
We then substitute the matched part of the line with this group using `\1` and then append our desired `/%u/`.
Effectively we have a long match pattern and are replacing only a subset of what was matched.
Each new set of `\(\)` creates a new group with an increasing number between 1-9.
We can nest these groups if desired, but we cannot have more than 9 automatically labeled groups.

By default `sed` will direct the output to STDOUT (eg your screen).
However we can use the `-i` flag to cause `sed` to do the replacement *inplace*, effectively editing the file.
Alternatively we can pipe the output to a new file using `> newfile`, or append to an existing file with `>> existingfile`.

Once I am confident that my replacements are going to wreck my files I run the following (in-place) substitution on all the `.sh` files in my directory:

```
sed -i -e 's:\(^#SBATCH.*\)/phancock/:\1/%u/:g' *.sh
```
{: .language-bash}

I find [this page](https://sed.js.org/) quite useful for testing my sed expressions on example text.
It has saved me from nerfing my files more than once.


## Bash as a programming language
The Bash shell acts as both a Unix shell (command line / command prompt / interactive terminal) and as a command language.
Bash is the default shell in most Linux distributions making it widely available.
You have no-doubt been using the Bash as a shell, especially since this is the preferred (and often only) way of interacting with HPC resources.
Learning more about the programming language side of Bash can help you to write better job scripts, scripts for your own quality of life improvements, and also make you more of a command line wizard.

Features of Bash as a programming language:
- control flow with `if/then/else/elif/fi` or `case/in/.../esac/`
- loops with `for/do/done` and `while/do/done`
- variable assignment with `var=value`
- variable lookup with `${var}` ( or just `$var` if you are lazy/brave)
- array assignment with `arr=(val1, val2, ...)`
- array access with `${arr[0]}`

Bash has a lot of built in commands which you can think of as functions.
A common design principle for Unix can be summarized as:
- Write programs that do one thing and do it well.
- Write programs to work together.
- Write programs to handle text streams, because that is a universal interface.

You should therefore think of the various Bash command line tools as functions that will modify streams of information.

Processes can emit output to one or both of STDERR (standard error) or STDOUT(standard out).
Standard practice is to put output into STDOUT and warning/error messages into STDERR.
When using `echo` the output will be directed to STDOUT.
By default both of these streams will be displayed in your terminal, however they can be redirected.
- redirection of STDOUT with `|`, `>`, and `>>`
  - `|` will *pipe* the output of one command to the input of another
    - example: `ls | wc -l` to count the number of lines generated by `ls`
    - example: `ls | more ` to show the results of `ls` one page at a time.
  - `>` will redirect the STDOUT of a command into a file, *overwriting* an existing file
    - example: `ls *.fits > image_files.txt`
  - `>>` will redirect the STDOUT of a command into a file, *appending* to an existing file
- redirection of STDERR with `2>`
  - example: `find . 2> /dev/null` will send all error messages to `/dev/null` (an information black hole)
  - example: `echo "Warning: Things are bad" >&2` will send the message to STDERR instead of STDOUT
  - example: `my_script.sh > ouptut.txt 2> log.txt` will save STDOUT into `output.txt` and STDERR into `log.txt`
  - example: `my_script.sh 2>&1 > all.txt` will redirect STDERR to STDOUT (`2>&1`) and then send both to the file `all.txt`


- process substitution with `<( )`
  - `cmd1 <(cmd2)` is equivalent to `cmd2 | cmd1`
  - If you need to do multiple pipes then process substitution can be handy
    - `
- process substitution with `>( )`
  - example: `my_script.sh > ouptut.txt 2> >(grep -e "^Warn" > warnings.txt)` will save STDERR to `warnings.txt` but only after passing it through `grep`.

When a bash command completes it will return an exit code.
By convention 0 means "success" and anything else means "error".
The exit code of the last completed command is stored in the special variable `$?`.
Note that this exit code is independent of what the program may or may not write to STDOUT or STDERR.
The `grep/sed` programs we looked at earlier will return 0 if a match is found and something else otherwise.

```
$ grep 'SBATCH' collect.sh 
#SBATCH --job-name=collect
#SBATCH --output=/fred/oz983/%u/collect_%A_out.txt
#SBATCH --error=/fred/oz983/%u/collect_%A_err.txt
#SBATCH --ntasks=1
#SBATCH --time=00:05
#SBATCH --mem-per-cpu=200

$ echo $?
0

$ grep 'other things' collect.sh 
$ echo $?
1
```
{: .output}

Confusingly, but also usefully, Bash interprets 0 as "true" and 1 and "false" in boolean comparisons.
Therefore, we can combine this with some short-cut binary logic functions to great effect:
- `ls myfile.sh &&  rm myfile.sh` will firstly test the truth of the left expression (try to locate a file using ls), and **only** if successful (exit code 0) it will then test the right expression (removing the file)
- `ls myfile.sh || echo "file not found"` will test (execute) the right expression only if the left expression is evaluated to be false

If you are writing your own bash scripts then you can set the exit code simply by using `exit 0` or `exit 1` (or whatever return code you want to use).


### Example bash script
The following pair of scripts `sfind.sh` and `aegean.sh` act as a wrapper around the `aegean` and `BANE` programs that are provided within the `aegean_main.sif` singularity container.

> ## sfind.sh
> This is the (template) job script that will be run by SLURM.
> It contains a basic set of SBATCH directives and two variables `base`/`image` that will be modified by another script.
> The user will not interact with this script directly.
> ~~~
> #! /bin/bash -l
> #SBATCH --export=NONE
> #SBATCH --time=10:00:00
> #SBATCH --nodes=1
> 
> base=BASEDIR
> image=IMAGE
> 
> # load singularity
> module load apptainer/latest
> # list my loaded modules (for debugging)
> module list
> 
> # define my container setup
> cont="singularity run -B $PWD:$PWD aegean_main.sif"
> # define some macros that run aegean/BANE from within the container
> aegean="${cont} aegean"
> BANE="${cont} BANE"
> 
> # Print commands and their arguments as they are executed (for debugging)
> set -x
> 
> # start a code block
> {
> 
> # move into the working directory
> cd ${base}
> 
> # look for the background/noise files
> if [ ! -e "${image%.fits}_bkg.fits" ] # Use the bash string substitution syntax
> then
>     ${BANE} ${image}
> fi
> 
> # process the image
> ${aegean} --autoload ${image} --table out.fits,out.csv
> 
> # for this code block:
> #  redirect STDERR/STDOUT to a subprocess that will prepend all lines
> #  with a date and time, and then redirect back to STDERR/STDOUT
> } 2> >(awk '{print strftime("%F %T")";",$0; fflush()}' >&2) \
>   1> >(awk '{print strftime("%F %T")";",$0; fflush()}')
> ~~~
> {: .language-bash}
>
{: .solution}

> ## aegean.sh
> This is the script that the user will interact with.
> It has a basic command line interface.
> ~~~
> #! /bin/bash
> function usage()
> {
> echo "obs_sfind.sh [-g group] [-d dep] [-q queue] [-M cluster] [-t] image
>   -g group   : group (account) to run as, default=oz983
>   -d dep     : job number for dependency (afterok)
>   -q queue   : job queue, default=<blank>
>   -t         : test. Don't submit job, just make the batch file
>                and then return the submission command
>   image      : the image to process" 1>&2;
> # ^ we send the help documentation to STDERR with 1>&2
> exit 1;
> # exit with status 1 (not ok)
> }
> 
> #default values for my arguments
> account="#SBATCH --account oz983"
> depend=''
> queue=''
> tst=
> extras=''
> 
> 
> # Parse option arguments.
> #    
> # Getopts is used by shell procedures to parse positional parameters
> # as options.
> # 
> # OPTSTRING contains the option letters to be recognised; if a letter
> # is followed by a colon, the option is expected to have an argument,
> # which should be separated from it by white space.
> while getopts 'g:d:q:M:t' OPTION
> do
>     case "$OPTION" in
> 	g)
> 	    account="#SBATCH --account ${OPTARG}"
> 	    ;;
> 	d)
> 	    depend="#SBATCH --dependency=afterok:${OPTARG}"
> 	    ;;
> 	q)
> 	    queue="#SBATCH -p ${OPTARG}"
> 	    ;;
> 	t)
> 	    tst=1
> 	    ;;
> 	? | : | h)
> 	    usage
> 	    ;;
>   esac
> done
> # set the obsid to be the last argument provided
> # by renaming the positional parameters
> shift  "$(($OPTIND -1))"
> image=$1
> 
> # Treat unset variables as an error when substituting.
> # Catches silly errors earlier on
> set -u
> 
> # if obsid is empty then just print help
> if [[ -z ${image} ]]
> then
>     usage
> fi
> 
> 
> # The working directory for our script
> base='/fred/oz983/phancock/'
> # The name of the script that we'll create and the location of the log files
> script="${base}queue/sfind_${image%.fits}.sh"
> output="${base}queue/logs/sfind_${image%.fits}.o%A"
> error="${base}queue/logs/sfind_${image%.fits}.e%A"
> 
> # build the sbatch header directives
> sbatch="#SBATCH --output=${output}\n#SBATCH --error=${error}\n${queue}\n${account}\n$> {depend}"
> 
>  # replace IMAGE and BASEDIR
> cat sfind.sh | sed -e "s:IMAGE:${image}:g" \
>                    -e "s:BASEDIR:${base}:g"  \
>                    -e "0,/#! .*/a ${sbatch}" > ${script}
> # ^appeend ${sbatch} after the first line matching the given pattern
> 
> # job invocation command, with a pause of 15s before the job starts
> sub="sbatch --begin=now+15 ${script}"
> 
> # if tst is not zero then
> # return the location of the script and the submission command
> if [[ ! -z ${tst} ]]
> then
>     echo "script is ${script}"
>     echo "submit via:"
>     echo "${sub}"
>     exit 0
> fi
> 
> # submit job and capture the output as a list
> jobid=($(${sub}))
> # output looks like "Submitted batch job 0123456"
> jobid=${jobid[3]}
> 
> # rename the err/output files as we now know the jobid
> error=$( echo ${error} | sed "s/%A/${jobid}/" )
> output=$( echo ${output} | sed "s/%A/${jobid}/" )
> 
> echo "Submitted ${script} as ${jobid}"
> echo "STDOUT is looged to ${output}"
> echo "STDERR is logged to ${error}"
> ~~~
> {: .language-bash}
>
{: .solution}

## WCSTools
According to the [README](http://tdc-www.harvard.edu/software/wcstools/wcstools.readme.html)
```
WCSTools is a set of software utilities, written in C, which create,
display and manipulate the world coordinate system of a FITS or IRAF
image, using specific keywords in the image header which relate pixel
position within the image to position on the sky.  Auxillary programs
search star catalogs and manipulate images.
```
{: .output}

WCSTools is not installed by default but you can download a copy from [http://tdc-www.harvard.edu/wcstools/](http://tdc-www.harvard.edu/wcstools/).
On ubuntu you can also install via `sudo apt install wcstools`.

Once again we have a powerful and many featured tool, of which a few functions are extremely useful, a few are available elsewhere, and some probably you'll never use.
I note this package here because it provides this functionality via the command line, making it easy (er) to write bash scripts that do smart things with fits files.

The functions which I find to be the most useful are:

| command | description                                                         |
| ------- | ------------------------------------------------------------------- |
| getfits | Extract portion of a FITS file into a new FITS file, preserving WCS |
| gethead | Return values for keyword(s) specified after filename               |
| sethead | Set header keyword values in FITS or IRAF images                    |
| getpix  | Return value(s) of specified pixel(s)                               |
| setpix  | Set specified pixel(s) to specified value(s)                        |
| gettab  | Extract values from tab table data base files                       |
| imhead  | Print FITS or IRAF header                                           |
| sky2xy  | Print image pixel coordinates for given sky coordinates             |
| xy2sky  | Print sky coordinates for given image pixel coordinates             |
| skycoor | Convert between J2000, B1950, galactic, and ecliptic coordinates    |

So for example I can write a script that will look at the header of a fits file and figure out how large the image is using:
```
imfile=1904-66_SIN.fits
xy=($(gethead ${imfile} NAXIS1 NAXIS2))
x=${xy[0]}
y=${xy[1]}
echo "File ${imfile} is ${x} by ${y} pixels in size"
```
{: .language-bash}



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