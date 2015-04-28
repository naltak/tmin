# Summary #

<font color='crimson' size='+1'><b>NOTE: This project is obsolete and replaced by the</b><i>afl-tmin</i> tool bundled with <a href='http://lcamtuf.coredump.cx/afl/'>american fuzzy lop</a>. **</font>**

_Tmin_ is a simple utility meant to make it easy to narrow down complex test cases produced through fuzzing. It is closely related to another tool of this type, [delta](http://delta.tigris.org), but meant specifically for unknown, underspecified, or hard to parse data formats (without the need to tokenize and re-serialize data), and for easy integration with external UI automation harnesses.

It also features alphabet normalization to simplify test cases that could not be further shortened.

# Functionality demo #

```
$ cat testcase.in
This is a lengthy and annoying hello world testcase.

$ cat testme.sh
#!/bin/bash

grep "el..*wo" || exit 0
exit 1

$ ../tmin -x ./testme.sh
tmin - complex testcase minimizer, version 0.03-beta (lcamtuf@google.com)
[*] Stage 0: loading 'testcase.in' and validating fault condition...
[*] Stage 1: recursive truncation (round 1, input = 53/53)
[*] Stage 1: recursive truncation (round 2, input = 27/53)
[*] Stage 1: recursive truncation (round 3, input = 14/53)
[*] Stage 1: recursive truncation (round 4, input = 10/53)
[*] Stage 1: recursive truncation (round 5, input = 8/53)
[*] Stage 1: recursive truncation (round 6, input = 7/53)
[*] Stage 2: block skipping (round 1, input = 7/53)
[*] Stage 2: block skipping (round 2, input = 6/53)
[*] Stage 2: block skipping (round 3, input = 5/53)
[*] Stage 3: alphabet normalization (round 1, charset = 5/5)
[*] Stage 3: alphabet normalization (round 2, charset = 5/5)
[*] Stage 4: character normalization (round 1, characters = 4/5)
[*] All done - writing output to 'testcase.small'...

== Final statistics==
 Original size : 53 bytes
Optimized size : 5 bytes (-90.57%)
Chars replaced : 1 (1.89%)
    Efficiency : 9 good / 49 bad
  Round counts : 1:6 2:3 3:2 4:1

$ cat testcase.small
el0wo
```

# Usage details #

The utility expects a file named `testcase.in` to be present in the current directory, and will write a minimal testcase to `testcase.small`. To optimize a test case for a target application, you can simply run:

`./tmin /path/to/program`

In this mode, _tmin_ will run /path/to/program in every cycle, feed a modified test case to program's stdin, and examine the exit status; the program exiting on a signal such as `SIGSEGV` will be interpreted as the test case still working, whereas a clean execution will be understood as the test case failing. You may also use a `-x` command-line switch to change the logic and treat non-zero return codes as fault conditions likewise, and `-w file` to save data to a specified location to be read by the tested application, instead of supplying it on stdin.

For remote testing, _tmin_ supports a `-s` command-line switch. In this mode, the behavior of the specified program is ignored, and the utility waits for `SIGUSR1` (clean execution) and `SIGUSR2` (fault condition) signal sent to _tmin_ process instead. Two common examples include:

`./tmin -s -w local_file.txt /bin/true`

`./tmin -s nc 127.0.0.1 1234`

As shown here, `nc` may be used as an easy wrapper for interaction with network services; and `/bin/true` may be used as a "decoy" target program when writing to local files.

In `-s` mode, the testing harness must prompt the tested application to read _tmin_ output, analyze the outcome, and then send an appropriate signal to the utility. An example of how to do all this when testing a HTML filter or other browser-based technology is given in `tmin/web-example` subdirectory.