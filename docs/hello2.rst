
Getting Started - Hello World!, part 2
======================================

Okay, so technically, none of the three hello scripts from :doc:`hello1` implement a true "Hello World!" script.
The main reason for that is that most RMS scripts will involve more than one computational step, and
so the hello scripts give an example of how to do that.  Second, the correct "Hello World!" involves
setting two RMS options to change its default behavior, and so it requires a bit more explanation.  The real
"Hello World!" script is shown at the bottom of this page.

Hello3 Script
-------------

The simplest of the three hello scripts is actually hello3, and that script is the following: ::

   #!/usr/bin/env rms
   
   ##argv=Arg

   #### hello Arg

   echo "Hello <Arg>, from the cluster!"

   #### helloAll all

   echo "Said hello to all of the arguments from the cluster!"

The first line of this script is the standard shell script shebang line used to make the file directly
executable, just as can be done with Perl or Python scripts.  In this case, the RMS command "rms" is
executed, and given the script to run.

The two most important lines of the script are the two that begin with "#### ": ::

   #### hello Arg
   #### helloAll all

Any line beginning with
exactly four '#' followed by a space is an RMS "step divider" line, and it marks the beginning of each
step of the pipeline script.  The format of those lines is "#### name column [exists]", where

    * "name" is the name of the step, and can be any string.  These names will be displayed in the
      progress messages that appear when a script is running.
    * "column" is either a column header from a column of the spreadsheet data, or the word "all"
      which tells RMS to run exactly one command for this step.  When RMS runs, it will create one
      executable command for each distinct value found on that column of the spreadsheet.
    * "exists" is an optional filename or path to a file.  If a name or path is given, that name/path
      is tested to see if it exists in the filesystem.  If it exists, then that command is skipped during
      the execution.

So, the two lines in the above script specify the two steps of the pipeline, one named "hello" which
is executed for each value in the "Arg" column of the spreadsheet data, and one named "helloAll"
which is executed once (i.e., for all of the spreadsheet data).

The lines after each of the step divider lines are the lines that are executed in the commands for
the step, and can be written in bash, perl, python or R.  Here, they are written in bash (the default),
and just use the echo command to write text to standard output.

The one unusual part of the first echo line is the "<Arg>" string appearing on the fourth line. ::

   echo "Hello <Arg>, from the cluster!"

That is an RMS "template element", which RMS replaces as it creates the executable command. The
basic format for a template element is "<column>" where "column" is a column header name from the
spreadsheet data.

If you have been wondering where the spreadsheet data is, that is the purpose of the second line,
"##argv=Arg".  In an RMS script, header text before the first step divider line can be used to
process the command-line arguments, and, in the header, a line of the form "##argv=column" tells RMS
to make a single column spreadsheet out of the command-line arguments, and use "column" as the
column header for that spreadsheet.

More details of the syntax and structure of RMS scripts can be found in :doc:`scriptbasics` and 
:doc:`scriptdetails`.

Hello2 Script
-------------

The script hello2 is very similar to that of hello3, but it contains one extra line, used to adjust the
behavior of RMS when it runs:  ::

   #!/usr/bin/env rms

   ##argv=Arg
   ##option=--log=-

   #### hello Arg

   echo "Hello <Arg>, from the cluster!"

   #### helloAll all

   echo "Said hello to all of the arguments from the cluster!"

In this script, the added line "##option=--log=-" is another RMS header section line, which sets RMS
command-line options.  In this case, the string "--log=-" is an RMS option that tells RMS where to
write each command's stdout and stderr text.  Setting --log to "-" tells RMS to direct the stdout
and stderr of the commands it runs to the stdout and stderr of the RMS program.  As a result, hello2
executes the same commands as hello3, but the command output is written (in order) to the screen.

Hello1 Script
-------------

The final script, hello1, is essentially the same as hello2, but it sets one additional RMS option, "-s",
telling RMS to perform a sequential execution of the commands on the current computer, instead of running
across the cluster: ::

   #!/usr/bin/env rms

   ##argv=Arg
   ##option="-s --log=-"

   #### hello Arg

   echo "Hello <Arg>!"

   #### helloAll all

   echo "Said hello to all of the arguments!"

(plus the text of the echo commands is slightly different).  For more information on the RMS command-line options,
see :doc:`clihelp`.

Hello World! Script
-------------------

Finally, the script that implements "Hello World!" is the following: ::

   #!/usr/bin/env rms

   ##option="-s --log=-"

   #### HelloWorld all

   echo "Hello World!"

or, you can run it from the command-line directly using ::

   rms -s --log=- -e 'echo "Hello World!"'
