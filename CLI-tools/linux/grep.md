# grep Cheatsheet
## Comparing Two Files
Pretend you have the following two files, called `file1.txt`:
```
one
two
three
four
4
five
```
and `file2.txt`:
```
one
two
three
four
4
five
six
6
two
```
### Comparing for Matches
```bash
grep -Fxf file1.txt file2.txt > matches.txt
```
Running the above command would produces a `matches.txt` that looks like this:
```
one
two
three
four
4
five
two
```
### Comparing for Diffs
```bash
grep -vFxf file1.txt file2.txt > diffs.txt
```
Running the above command would produce a `matches.txt` that looks like this:
```
six
6
```