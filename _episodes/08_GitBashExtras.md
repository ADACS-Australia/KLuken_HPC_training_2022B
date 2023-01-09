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
- branching and merging
- sync a project to two repositories
### CI/CD
- writing tests
- running tests locally
- running tests on github
- auto documentation


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
- `grep 'a(b|c)\1' text.dat` will match abb or acc but not abc. The \1 
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


## Bash as a programming language
- sub-shells / arrays
- substrings
  - using % and #
- making a cli with --help
- piping with `|`, `>`, and `>>`
- redirecting STDIN/STDOUT/STDERR
    - time stamping logfile lines


### environment management
- .bash_rc etc
  - what they do and how to use them
- venv/conda environments

## X11
- how to open gui apps remotely
- link/discuss nomachine