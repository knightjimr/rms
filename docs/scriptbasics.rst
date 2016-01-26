
Writing RMS Scripts - Basics
============================

An RMS script is a text file, just like any Perl or Python script.  And the bulk of the script itself will
actually be written in bash, Perl, Python or R (whichever language you choose to write each step in).  The
RMS syntax involves three elements that define the RMS-specific parts of the script:

* "Step divider" lines that begin with "#### " (four pound signs followed by a space).  These lines
  separate the steps of the pipeline.
* Option lines that begin with "##word" or "##word=", used to define optional values for the header or
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

:doc:`scriptdetails` describes the other optional section that can be included in RMS scripts, and goes into
complete detail about the features of each section.

Header Section
--------------

The header section occurs before the first step divider line (and before the environment section), and is
used to either hard-code some input for RMS, or to provide a script that should be run to process the
command-line arguments.  The examples in :doc:`hello2` contained header sections with hard-coded RMS directives,
and the four most common directives that can be included in the header are the following:

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
  given in the RMS script header.

* User-defined, name-value lines of the form "##name=( values )", where the "name" can then be used in
  template elements in the script, and are replaced with the value or values found within the parentheses.
  This is commonly used to define a option value to the rest of the script (like the genome to be used),
  or create a set of values to generate commands for (like a range of k-mer values to test when aligning
  reads): ::

     ##genome=( hg19 )
     ##kmer=( 10 12 14 16 18 20 22 24 26 28 30 )

Each of these directive lines must occur at the beginning of the line, and must be of the form "##name", 
"##name=value" or "##name=(...)", and may not contain spaces, except between the parentheses (this is
to limit the interference with real Python or Perl program comment lines, so that they are not mistaken
for RMS lines).  The full set of header directives is described in :doc:`scriptdetails`.

The header section may also contain a bash, Python, Perl or R script, which is used to allow the script to
process the command-line arguments, instead of RMS.  If there is at least one line in the header that is not
a RMS directive line, and is not a comment line beginning with "#", then RMS assumes that the header
contains a script, and will pass it the command-line arguments for processing.

What this header section script needs to do is to output RMS directive lines on its standard output, which is
piped into RMS (when it executes the script) to define the input for the program.  For example, a Python
script which implements the "##argv=Arg" functionality is the following: ::

   ##python

   import sys

   print "##sheet=EOF"
   print "Arg"

   for arg in sys.argv[1:]:
      print arg

   print "EOF"

The first line of this script is "##python", telling RMS that the language for the script is Python (there is
also "##perl", "##R" and "##bash" for those languages, although the default is bash, so "##bash is not
necessary).  The rest of the script is Python code which outputs the line for a "##sheet" directive, defining
a one-column spreadsheet (with column header "Arg") containing the command-line arguments.

Any functionality is permitted in this script.  You can also read files, use subprocess to call commands,
whatever is necessary to parse the command-line arguments and generate the spreadsheet data and options
to be used in the RMS execution).  Once this script terminates, RMS will process the directives and begin
the execution.

Environment Section
-------------------

Just as with cluster jobs that you submit, the commands that RMS execute across the cluster do not begin
with the environment (PATH, LD_LIBRARY_PATH, ...) values that exist when you execute the RMS command, as
many clusters do not support inheriting the environment variables.  RMS takes case of loading your .bash_profile
and .bashrc files for the commands (no need for "source ~/.bashrc" in your scripts), and also ensures that
the command executes from the same current working directory as when you started the RMS command (so, no need
for "cd /my/hardcoded/starting/directory" in your scripts either).  But, it may not have the other
environment variables, and, in particular for writing scripts to be run by other users, there may not be
an assurance that the software you want to run in the RMS script is already setup in the users' environment.

The environment section is used to setup the environment variables for each commands' script execution.
It begins with a "##env" line before the first step divider line, and all of the lines between "##env" and
the first step divider line are assumed to be the environment section.

For example, if you want to write an RMS script to use samtools to index one or more bam files, but are not
sure that the samtools executable is on each users' PATH (but you know the executable is in
/opt/bioinfo/software/samtools-1.2), then the following script will ensure that the samtools executable is
found for each execution of the command:  ::

    ##argv=file

    ##env
    export PATH=/opt/bioinfo/software/samtools-1.2:$PATH

    #### index file -
    samtools index <file>

or, if your system has the module software, you can just put "module samtools-1.2" in the environment section,
and it will get loaded before the lines of the "index" script execute.

Environment sections are also used for Python, Perl or R scripts.  When RMS creates an executable command, it
creates a bash script that contains (1) RMS initialization lines, (2) the environment section lines and (3)
xxx.  For RMS steps whose language is bash, RMS just added those lines to the bash script. For the other
languages, those lines are written to a language-specific script, and the bash script contains a launcher
to run that language-specfic script.  So, regardless of the language of the step, the environment section
will be loaded for the commands of that step.

Step Section
------------

