
Writing RMS Scripts - Basics
============================

An RMS script is a text file, just like any Perl or Python script.  And the bulk of the script
itself will actually be written in bash, Perl, Python or R (whichever language you choose to write
each step in).  The RMS parts are three elements that define the RMS-specific parts of the script:

* "Step divider" lines that begin with "#### " (four pound signs followed by a space).  These lines
  separate the steps of the pipeline.
* Option lines that begin with "##word" or "##word=", which are used to define optional values for the header or
  steps of the pipeline. The specific allowed values of "word" are different for each section of the
  RMS script.
* Template elements of the form "<column>" or "<column,option...>" in the text of the steps, which
  define the fill-in-the-blank elements that RMS replaces when creating each executable command that
  is run.

An RMS script can be as simple as a single step divider line followed by the script for that step, but
typically there are up to three common sections in most scripts:

* An optional "header" section which processes the command-line arguments and/or provides hard-coded
  input to RMS.
* An optional "environment" section which is typically used to setup the environment variables ($PATH,
  ...) for each of the commands run by RMS.
* One or more "step" sections, which are the steps of the script.

:doc:`scriptdetails` describes the other optional sections that can be included in RMS scripts, and goes into
complete detail about the features of each section.

Header Section
--------------

The header section occurs before the first step divider line (and before the environment section),
and is used either to hard-code some input for RMS or to provide a script that should be run to
process the command-line arguments.  The examples in :doc:`hello2` contained header sections with
hard-coded RMS directives, and the four most common directives that can be included in the header
are the following:

* The "argv" directive to tell RMS to convert the command-line arguments into a single column spreadsheet
  with the given column header name: ::

     ##argv=name

* The "sheet" directive to give RMS a hard-coded spreadsheet of values.  The format of the sheet directive
  is similar to how bash script lines can be passed as standard input to a program, using a marker like "EOF"
  at the beginning and end of the text.  An example here is the following: ::

     ##sheet=EOF
     Sample File Type
     S1 S1_reads_R1.fastq R1
     S1 S1_reads_R2.fastq R2
     S2 S2_reads_R1.fastq R1
     S2 S2_reads_R2.fastq R2
     EOF

  The lines between "##sheet=EOF" and "EOF" contain the spreadsheet data that RMS will parse and include
  in its input (in this case, a three column spreadsheet with "Sample", "File" and "Type" columns).

* The "option" directive to pass RMS command-line options.  These options will be parsed as would options
  actually occurring on the command-line, however options found on the command-line will override options
  given in the RMS script header. ::

     ##option="-s --log=-"

* User-defined, name-value lines of the form "##name=( values )", where the "name" can then be used in
  template elements in the script, and are replaced with the value or values found within the parentheses.
  This is commonly used to define a option value to the rest of the script (like the genome to be used),
  or create a set of values to generate commands for (like a range of k-mer values to test when aligning
  reads): ::

     ##genome=( hg19 )
     ##kmer=( 10 12 14 16 18 20 22 24 26 28 30 )

The full set of header directives is described in :doc:`scriptdetails`.  Each of these directive
lines must occur at the beginning of the line, and must be of the form "##name", "##name=value" or
"##name=(...)", and may not contain spaces, except between the parentheses or the quotes.  The
reason for this is to limit the interference the RMS line format may have with real Python or Perl
program comment lines, so that they are not mistaken for RMS lines.

Header Script
^^^^^^^^^^^^^

The header section may also contain a bash, Python, Perl or R script, which is used to allow the
script to process the command-line arguments, instead of RMS.  If there is at least one line in the
header that is not a RMS directive line, and is not a comment line beginning with "#", then RMS
assumes that the header contains a script, and will pass it the command-line arguments for
processing.

What this header section script needs to do is to output RMS directive lines on its standard output,
which is piped into RMS to define the input for the program.  For example, a Python script which
implements the "##argv=Arg" functionality is the following: ::

   ##python

   import sys

   print "##sheet=EOF"
   print "Arg"

   for arg in sys.argv[1:]:
      print arg

   print "EOF"

The first line of this script is "##python", telling RMS that the language for the script is Python (there is
also "##perl", "##R" and "##bash" for those languages).  The rest of the script is Python code which outputs
the line for a "##sheet" directive, defining a one-column spreadsheet (with column header "Arg") containing
the command-line arguments.

Any functionality is permitted in this script.  You can also read files, use subprocess to call commands,
whatever is necessary to parse the command-line arguments and output the spreadsheet data and options
to be used in the RMS execution (on its standard output).  Once this script terminates, RMS will process
the directives and begin the execution.

Environment Section
-------------------

Many clusters don't support the inheritance of environment variables (PATH, PWD, ...) for the jobs
that are submitted, so the commands that RMS executes across the cluster may not begin with the
environment values that exist when you execute the RMS command.

RMS takes care of loading your ~/.bash_profile and ~/.bashrc files (so, no need for "source
~/.bashrc" in your scripts), and also sets the current working directory for the command to be the
same as when you started the RMS command (so, no need for "cd /my/hardcoded/starting/directory" in
your scripts either).  But, it may not have the other environment variables, and, in particular for
writing scripts to be run by other users, there may not be an assurance that the software you want
to run in the RMS script is already setup in the users' environment.

The environment section is used to setup the environment variables for each commands' script execution.
It begins with a "##env" line before the first step divider line, and all of the lines between "##env" and
the first step divider line are assumed to be the environment section.

For example, if you want to write an RMS script to use samtools to index one or more bam files, but
are not sure that the samtools executable is on each user's PATH (but you know the executable is in
/opt/bioinfo/software/samtools-1.2), then the following script will ensure that the samtools
executable is found for each execution of the command: ::

    ##argv=file

    ##env
    export PATH=/opt/bioinfo/software/samtools-1.2:$PATH

    #### index file -
    samtools index <file>

Whatever lines you would normally put at the beginning of a bash script to setup the environment can be put
here, and it will get loaded at the beginning of every command execution.

Environment sections are also used for Python, Perl or R scripts.  When RMS creates an executable
command, it creates a bash script that contains (1) RMS initialization lines, (2) the environment
section lines and (3) a language-specific body.  For RMS steps whose language is bash, RMS just adds
the lines from the RMS step directly into the bash script.  For the other languages, the bash script
contains a launcher which runs python, perl or Rscript on a file containing the lines from the RMS
step.

Step Section
------------

The rest of the RMS script are the "step sections" that make up the pipeline steps to be executed.
Each section begins with a step divider line and RMS option lines.  The rest of the section is a
bash, python, perl or R script, written (with two exceptions) just as they would be written as a
separate script.  The first exception is that the script can contain "template elements",
fill-in-the-blank elements like "<file>" that RMS will replace when it generates the command script.
The second exception is that any lines occurring in the environment section (described above) do not
need to be included in each step.

The step divider line that begins a step section serves three purposes, (1) mark the beginning of a
new step, (2) define what commands to generate for the step and (3) support incremental execution
with a "file test" to determine when to skip the execution of the step.  The format of the step
divider line is the following: ::

   #### name column(s) [filetest]

The "name" value is the name of the step, is shown in progress output and error messages, and can be
any non-whitespace string.  The "column(s)" value is a comma-separated list of the spreadsheet
column headers which will be used to determine what commands are generated for this step during
execution.  The optional "filetest" value is a filename or path to be checked for existence during
execution.  If that file/path does exist when that command is ready to be run, the command is
skipped (not run) as part of the execution.

The "column(s)" value is what RMS uses to determine what commands will be generated for the pipeline
step.  RMS takes the spreadsheet data given as input, finds the distinct sets of values in those
columns of the spreadsheet, and will create one command for each distinct set.  So, if the column(s)
value is "Sample", one command will be created for each sample in the column.  If it is
"Sample,File", one command is created for each distinct pair of Sample and File values.  If the
column value is the special keyword "all", then RMS will create one command for the step (covering
"all" of the spreadsheet data).  If a name-value line was defined in the header of the script, that
can also be used as a column(s) value, so if this was in the script header: ::

   #kmer=( 15 20 25 30 35 40 45 50 )

then a column(s) value of "Sample,kmer" will create one command for each pair of samples and kmers.

As part of this computation, the rows of the input spreadsheet data are partitioned into the sets of
rows for each distinct set of values found (i.e., each command generated for the step will use the
spreadsheet rows that contain those specific column(s) values).  These rows are what will be used
when performing the replacement of the template elements in the body of the step (see below).


Step Options
^^^^^^^^^^^^

The step option lines are used to tell RMS what resources (cores, memory, tmp space) will be used
during the computation of the step commands, as well as telling RMS about the step language and
parallelism/recovery.  

There are four resource option lines, each of takes a number as its value (except for ##tmp, which
may also take a path, see scriptdetails).  They are the following: ::

   ##ppn=5       [number of cores, default 1]
   ##io=3        [i/o concurrency limit, default no limit]
   ##mem=40      [main memory limit in GB, default 10]
   ##tmp=60      [tmp space limit in GB, default 10]

These are not hard limits, but help RMS run the commands on the compute nodes quickly and safely
(too high a cpu load or too much I/O will slow your computation down significantly, using too much
memory on a compute node has been the cause of nearly every "mysterious" crash of a cluster job, in
my experience, and running out of disk space is usually an unrecoverable error...so making an effort
to avoid these issues will make your script faster and more robust).

The ##io limit restricts how many commands from the step can run at the same time on the same
compute node, in order to avoid thrashing on the I/O channel (as the compute node tries to satisfy
all of the reads and write the commands are making).  To get a sense of the problem, try running the
following the script with and without the ##io line on a compute node (where you can substitute the
samtools sort command for a command you know is heavily I/O bound), making sure you are running at
least as many commands as cores on the compute node, i.e., like "rms -s sort.rms *.bam": ::

   ##argv=bam
   #### sort bam -
   ##io=3
   samtools sort <bam> > <bam>.sorted.bam

Particularly with nodes that have 10-20 cores, the running time without the ##io line should be much
longer than with (as the overall execution keeps within the capability of the compute node's and the
NFS I/O hardware, so that the file contents can be served up efficiently).

The other option lines commonly used are the lines defining the language for the step: ::

   ##python, ##perl, ##R or ##bash

as well as two option lines, ##after and ##redo, that allow for step parallelism and error
recovery.  The "##after=..." takes a comma-separated list of step names (these steps must be defined
earlier in the RMS script), and tells RMS that this step can execute immediately after those steps
complete, instead of when the previous step in the script completes.  For example, the
"scatter-gather" pattern can use step parallelism with a script as follows: ::

   ...
   #### step1 sample
   ...

   #### step2 sample,file
   ...

   #### step3 sample,file
   ##after=step1
   ...

   #### step4 sample
   ##after=step2,step3
   ...

(Note that this script performs both step parallelism, using the ##after option, and data
parallelism, using "sample,file" column-based parallelism for step2 and step3.)

The ##redo option line takes a value which is the number of times the command should be restarted
afte an error, along with an optional command to run when restarting the command (i.e., to clean up
temp files or reset necessary files/values):  ::

   ##redo=1
   ##redo=2;rm -f <sample>/tmp_*.bam

RMS will restart any step command that failed to complete (i.e., returned a non-zero exit status).
If the command fails the given number of times, the command will be considered as failed.

Step Lines
^^^^^^^^^^

The remaining lines of the step 
Step lines, with template elements and rms commands (rmssync, rmscp, rmslock).

<column>
<column,glob=True>
