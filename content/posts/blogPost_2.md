+++
title = 'Using grep,awk,sed and other tools to parse data'
date = 2024-12-16T12:26:34-05:00
author = "Harsha"
draft = false
tags = ["linux","bash","grep","sed","awk"]
+++


In the simplest terms, grep (global regular expression print) will search input files for a search
string, and print the lines that match it.

Beginning at the first line in the file, grep copies a line into a
buffer, compares it against the search string, and if the comparison passes, prints the line to the
screen. Grep will repeat this process until the file runs out of lines. Notice that nowhere in this
process does grep store lines, change lines, or search only a part of a line.

## Eample data for this article

```bash
cat << EOF >> a_file
boot
book
booze
machine
boots
bungie
bark
aardvark
broken$tuff
robots
EOF

```

## A simple example for grep

A simple example for grep:

```bash
grep "boo" a_file
```

This will print all lines in a_file that contain the string "boo".
The result looks like this:

```bash
boot
book
booze
boots
```

### Useful Options

To print the line number of the matching lines, use the -n option:

```bash
grep -n "boo" a_file
```

To print the file name of the matching lines, use the -l option:

```bash
grep -l "boo" a_file
```

To switch the search results that are complimentary to the search string, use the -v option:

```bash
grep -v "boo" a_file
```

To display number of lines that match the search string, use the -c option:

```bash
grep -c "boo" a_file
```

To show additional after the matching lines, use the -A options:

```bash
grep -A2 "boo" a_file
```

To show additional lines before lines, use the -B options:

```bash
grep -B2 "boo" a_file
```

To show additional lines before and after the matching lines, use the -C options:

```bash
grep -C2 "boo" a_file
```

## Regex

A regular expression is a compact way of describing complex patterns in text. With grep, you can
use them to search for patterns.

You may use grep to search using basic regexps such as to search the file for
lines ending with the letter e:

```bash
grep -E "e$" a_file
```

Or to search for lines that contain the word "boo" followed by any character:

```bash
grep -E "boo." a_file
```

Or to search for lines that contain the word "boo" followed by any character, but not the letter "e":

```bash
grep -E "boo[^e]" a_file
```

If you want a wider range of regular expression commands then you must use 'grep -E' (also known
as the egrep command). For instance, the regexp command ? will match 1 or 0 occurences of the
previous character:

```bash
grep -E "boots?" a_file
```

 combine multiple searches using the pipe (|) which means 'or' so can do things like:

 ```bash
 grep -E "boo|bark" a_file
 ```

## AWK

awk is a scripting language used for manipulating data and generating reports.

### Basics

An awk program operates on each line of an input file. It can have an optional BEGIN{} section of
commands that are done before processing any content of the file, then the main {} section works
on each line of the file, and finally there is an optional END{} section of actions that happen after
the file reading has finished

BEGIN { …. initialization awk commands …}
{ …. awk commands for each line of the file…}
END { …. finalization awk commands …}

These
'pattern-matching' commands can contain regular expressions as for grep. The awk commands can
do some quite sophisticated maths and string manipulations, and awk also supports associative
arrays.

| Input statement                          | output statement                                                                                                                                                                                                                           |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| close(file [, how]) close(file [, how])  | Close file, pipe or co-process.                                                                                                                                                                                                            |
| getline                                  | Set $0 from next input record.                                                                                                                                                                                                             |
| getline <file                            | Set $0 from next record of file.                                                                                                                                                                                                           |
| next                                     |  Stop processing the current input record. The next<br>input record is read and processing starts over<br>with the first pattern in the AWK program. If the<br>end of the input data is reached, the END block(s),<br>if any, are executed.|
| print                                    | Prints the current record.                                                                                                                                                                                                                 |
| print expr [, expr ...]                   | Prints the values of the expressions.|

### AWK string functions

| Function                         | Description                                                                                                                                                                                                                          |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|gsub(r, s [, t]) | Replace (g)sub(st)ring. |
|index(s, t) | Return starting index of t in s, or 0 if not found. |
|length(s) | Return length of s. |
|match(s, r) | Match regular expression r in s. |
|split(s, a [, r]) | Split s into parts separated by r, put results in array a. |
|sprintf(format, expr) | Return a string formated according to format, using the expressions. |
|sub(r, s [, t]) | Substitute (sub)string. |
|substr(s, i [, n]) | Return the substring of s starting at i of length n. |
|tolower(s) | Return a lower case version of s. |
|toupper(s) | Return an upper case version of s. |

### AWK math functions

| Function                         | Description                                                                                                                                                                                                                          |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| atan2(y, x) | Return the arctangent of y/x. |
| cos(expr) | Return the cosine of expr. |
| exp(expr) | Return e (the base of natural logarithms) raised to the power of expr. |
| int( expr) | Return the integer part of expr. |
| log(expr) | Return the natural logarithm of expr. |
| rand() | Return a random number between 0 and 1. |
| sin(expr) | Return the sine of expr. |
| sqrt(expr) | Return the square root of expr. |
| srand([ expr]) | Seed the random number generator. |

### AWK built-in variables

| Variable                         | Description                                                                                                                                                                                                                          |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ARGC | The number of command-line arguments. |
| ARGV | The command-line arguments. |
| ENVIRON | An associative array of name/value pairs representing the current environment. |
| FILENAME | The name of the current input file. |
| FNR | The number of the current record in the current file. |
| FS | The input field separator. |
| NF | The number of fields in the current record. |
| NR | The number of the current record. |
| OFS | The output field separator. |
| ORS | The output record separator. |
| RLENGTH | The length of the string matched by match. |
| RS | The input record separator. |
| RSTART | The starting position of the string matched by match. |
| SUBSEP | The value of the subscript separator. |

### AWK command line and usage

Pass varianbles to awk from the command line.

```bash
awk -v skip=3 '{for (i=1;i<skip;i++) {getline}; print $0}' a_file
```

### AWK script file

```bash
#!/usr/bin/awk -f
BEGIN {skip=3}
{for (i=1;i<skip;i++)
 {getline};
print $0}
}

```

## SED and regex

What if one of the characters you wish to use in the search command is a special symbol, like '/'
(e.g. in a filename) or '*' etc? Then you must escape the symbol just as for grep (and awk). Say you
want to edit a shell scripts to refer to /usr/local/bin and not /bin any more, then you could do this.

````bash
sed 's/\/bin/\/usr\/local\/bin/g' myscript > newscript
````

What if you want to use a wildcard as part of your search – how do you write the output string? You
need to use the special symbol '&' which corresponds to the pattern found. So say you want to take
every line that starts with a number in your file and surround that number by parentheses:

````bash
sed 's/^[0-9]/(/&)/g' myfile
````

### other sed examples

The general form is
sed -e '/pattern/ command' my_file
where 'pattern' is a regexp and 'command' can be one of 's' = search & replace, or 'p' = print, or 'd' =
delete, or 'i'=insert, or 'a'=append, etc. Note that the default action is to print all lines that do not
match anyway, so if you want to suppress this you need to invoke sed with the '-n' flag and then you
can use the 'p' command to control what is printed. So if you want to do a listing of all the
sub-directories you could use

```bash
ls -l | sed -n -e '/^d/ p'
```

as the long-listing starts each line with the 'd' symbol if it is a directory, so this will only print out
those lines that start with a 'd' symbol.
Similarly, if you wanted to delete all lines that start with the comment symbol '#' you could use

```bash
sed -e '/^#/ d' my_file
```

i.e. you can achieve the same effect in different ways!
You can also use the range form

```bash
sed -e '1,100 command' my_file
```

to execute 'command' on lines 1,100. You can also use the special line number '$' to mean 'end of
file'. So if you wanted to delete all but the first 10 lines of a file, you could use

```bash
sed -e '11,$ d' my_file
```

You can also use a pattern-range form, where the first regexp defines the start of the range, and the
second the stop. So for instance, if you wanted to print all the lines from 'boot' to 'machine' in the
a_file example you could do this:

```bash
sed -n -e '/boot$/,/mach/p' a_file
```

which will then only print out (-n) those lines that are in the given range given by the regexps.
