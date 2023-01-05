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
Furthermore we can allow the line to start with some indentation of space/tab combo: `
grep '^\s*# ' *.py *.sh *.c`.
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




## wcstools
command line tool for easy get/set of data/metadata in fits format

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