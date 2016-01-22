
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

Header Sections
---------------

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


