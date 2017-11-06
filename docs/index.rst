.. RMS documentation master file, created by
   sphinx-quickstart on Thu Jan  7 13:37:39 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

RMS - Run My Samples
====================

RMS is a “cluster scripting language” and execution engine, making the creation of computational
pipelines, and running them across a compute cluster, easy to do.  The software takes a templated
RMS script plus spreadsheet data files, generates commands by using the spreadsheet data to fill in
the templates (i.e., for each file, each sample, each trio), and runs them on the current computer
or across a cluster.

And, calling it a language overstates the case just a bit.  It really consists of extra lines to
organize the Bash, Perl, Python or R scripts of a pipeline, along with fill-in-the-blank “template
elements” that are replaced when creating the executed commands.  You write the pipeline steps in
any combination of the four languages, include the RMS lines and template elements, and then run RMS
with the script and the spreadsheet data as you would any program.  It handles all of the details of
creating, queueing and running jobs across the cluster.

For example, if you have the file 'myfiles.txt', containing the names of three files plus a column
header on the first line, ::

   File
   myfile1
   myfile2
   myfile3

and have the following RMS script 'myscript.rms', where the first line is an RMS line saying to
execute the lines below as a command (and to execute that for each "File" in the spreadsheet data), ::

   #### runTheFiles File
   mycommand <File>

then you can run rms as follows: ::

   rms myscript.rms myfiles.txt

and RMS will execute the three commands "mycommand myfile1", "mycommand myfile2" and "mycommand
myfile3" across your cluster, handling all of the details of cluster jobs and queues and qsub's.

The best place to get started is with :doc:`installation`, because you likely will need to add a
configuration file describing your cluster, even if the RMS software has been installed already.
:doc:`hello1` will let you test the software on your cluster and see another simple RMS example.
Then, the rest of the documents contain more examples and go into detail about RMS scripts and
running RMS.


.. toctree::
   :maxdepth: 1

   installation
   hello1
   hello2
   scriptbasics
   clihelp

