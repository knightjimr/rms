
RMS Script Format
=================

Sections are:

* Setup/Command-Line Section
* Environment Section
* Language Initialization Section
* Step Section
    - Step Divider Line
    - Step Options
    - Step Body
    - Template Element Format

Setup/Command-Line Section
--------------------------

* ##python, ##perl, ##R or ##bash
* ##option="..." or #option=...
* ##lang=...
* ##sheet=EOF ... EOF
* ##argv=...
* ##redo=...
* ##totalio=...
* ##outline=...
* ##errline=...
* ##...=(...)

Environment Section
-------------------

Bash-only.  Goes at beginning of every command script.  Template element substitution occurs in this section.

##local=...

Lanuage Initialization Section
------------------------------

Starts with "##init ...", where "..." is "bash, "perl", "python, "R".

Each script in that language will begin with lines in this section.  Multiple sections for different languages
allowed, one section per language.

Step Section
------------

Structure of step section.  Template element substitution occurs in each section.

* Step Divider Line
* Step Option Lines
* Step Body


Step Divider Line
^^^^^^^^^^^^^^^^^

Format structure

   * "#### "
   * "stepName"
   * column or columns (comma-separated, no spaces allowed)
   * Path or "-" for existence test (optional, single file only)

Columns must be spreadsheet columns or variables.  Multiple columns allowed, separated by commas,
no whitespace allowed, or the word "all" for all of the data.
RMS creates one command for each distinct value in that column of the input
data (or each distinct set of values in the columns of data).

Step Option Lines
^^^^^^^^^^^^^^^^^

Describe the resources for the step and/or set the options for the step's execution.

* ##python, ##perl, ##R or ##bash - step language, default is ##lang= default or ##bash
* ##after=... - step dependency, this step runs after list of steps comma-separated list of previous steps)
* ##ppn=... - number of cores needed by this step
* ##io=... - maximum number of this step's commands to be run on a single node (for I/O limitations)
* ##mem=... - GB of memory needed by this step
* ##tmp=... - GB of local disk space neede by this step (number or 3x<path> string)
* ##redo=... - number of times to retry the command on failure, with optional reset command
* ##name=column if column=value - set a new RMS variable called "name" to be equal to the set of distinct values where "column=value" is True in the spreadsheet data
* ##outline=... - limit number of lines reported from each command's stdout
* ##errline=... - limit number of lines reported from each command's stderr


Step Body
^^^^^^^^^

Written in the language defined by the step option lines or the default language option.
Describe command generation and command script organization (bash script with launcher for
python/perl/R script).  Line numbers and error messages.  Bash scripts die on first error (careful with 
status logic).  RMS commands rmscp, rmssync, rmslock.

Template Element Format
^^^^^^^^^^^^^^^^^^^^^^^

Basic template element "<column>", where column is a column or RMS variable in the input data.
No whitespace allowed, except as defined in the options below.  There are also defined template elements
from the environment setup for a command:

* <ppn> - Current step's ppn option value
* <tmp> - Temp directory created for the command's execution (separate for each command, with auto cleanup)
* <local> - Temp directory created for each compute node (shared across all commands executed on the node),
            loaded by ##local RMS lines
* <script>, <bin>, <bin..>

RMS performs a string replacement of <column> with the value or distinct set of values corresponding
to column (as defined by the step set of distinct values).  If multiple values found, default replacement
separates values by a single space.

Options can appear as comma-separated strings after the column name.

* glob=true|false - use globbing of all non-whitespace text around element for multiple value replacement
* sep=',' - use the given separator for multiple values, instead of a space
* quote='"' - surround each value by the given quote character, after globbing and prefix/suffix addition
* prefix="..." - add the given string before each value, whitespace permitted within quotes
* suffix="..." - add the given suffix after each value, whitespace permitted within quotes

Examples of element replacement.  If the distinct values of column "sample" are the values "me", "my" and "mine",
then the following template replacements occur

   "ls <sample>" -> "ls me my mine"
   "rm <sample>.bam" -> "rm me my mine.bam"  (likely not what you want)
   "rm <sample,glob=True>.bam" -> "rm me.bam my.bam mine.bam"
   "myscript <sample,prefix="-V ",suffix=".bam">" -> "myscript -V me.bam -V my.bam -V mine.bam"
   "samples = [ <sample,quote='"',sep=", "> ]" -> "samples = [ "me", "my", "mine" ]"  (useful for python)

Recursive replacement is supported, but each replacement operation occurs separately. If column "project"
is defined as the single value "prj", then the following replacements occur:

   "ls <project>/<sample>.bam" -> "ls prj/me my mine.bam"  (likely not what you want...)
   "ls <project>/<sample,glob=True>.bam" -> "ls prj/me.bam prj/my.bam prj/mine.bam"
   "ls <sample>/<sample>.bam" -> "ls me my mine/me my mine.bam" (likely not what you want...)

For this last example, there is not currently an RMS way to get what you want,
namely "ls me/me.bam my/my.bam mine/mine.bam", because each replacement occurs separately.

Language-Specific Template Element Tips
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For each of the four languages (bash, python, perl and R), here are examples of how you can (1)
assign a template elements values to a variable, (2) perform an if test on a single value
element and (3) loop over the values of a template element.  These should be helpful building
blocks to communicating between RMS and the step script.

PROJECT="<project>"
echo $PROJECT

SAMPLE=( <sample,quote='"'> )
echo ${SAMPLE[1]}

if [ "<project>" == "prj" ] ; then
   echo This is the prj project.
else
   echo This is not the prj project.
fi

for sample in <sample,quote='"'> ; do
   echo $sample
done


project = "<project>"
print project

sample = [ <sample,quote='"',sep=','> ]
print sample[0]

if "<project>" == "prj":
   print "This is the prj project."
else
   print "This is not the prj project."

for sample in [ <sample,quote='"',sep=','> ]:
    print sample




