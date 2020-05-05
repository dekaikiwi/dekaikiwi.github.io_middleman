---
title: Useful grep options
date: 1588695397
---
# grep

The following are my notes and thoughts going through the manual page for grep. 

The following options are what I found to be most useful to myself personally. 
## Learnings...

-E vs -G (Extended vs Basic Regexp

While in GNU grep there is no no difference between the extended and basic group of Regular Expressions. In other versions the basic group is less powerful than the extended group. 

Therefore when using regular rexpressions it would be a good idea for compatablity to specify the `-e` option

## Options

-F - Useful if you are looking for a string that would be interpreted differently in Regex.

-i (--ignore-case) - Ignore case. Probably should be set in most cases?

-v (--invert-match) - Invert Match, everything that wouldn't usually matches is now a match and every thing that would have matched does not.

-c (--count) Only print number of matching lines. Maybe useful for bash scripts?

-H (--with-filename) Already default when more than one file is searched. Q: Is this default behavior for all versions of grep?

-n (--line-number) Prefix each line with the number of the match within the input file. A useful option for most situations.

-r (--recursive) Read all files under each directory recursively.



### Getting extra context

The following options help get an idea about what text is around matching lines.

-A (--after-context) Prints the set number of lines of trailing context after the matching lines

-B (--before-context) Prints the set number of lines of leaading context before matching lines

-C (--context) Prints n number of lines of context of either side of a matching line.

## Default grep command

The following would be a good general use grep command.

```
grep -an -e <PATTERN> <FILE>
```

Looks for matches and also...

- Displays the line number of every match.
- Look for case insenstive matches


### Recursive Option

```
grep -ran <Pattern> <FILE>
```

## Note for Bash Scripts

grep will exit with status 0 if a line was selected, 1 if no lines were selected and 0 if an error occured.
