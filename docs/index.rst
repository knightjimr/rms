.. RMS documentation master file, created by
   sphinx-quickstart on Thu Jan  7 13:37:39 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

RMS - Run My Samples
====================

RMS is a “cluster scripting language” and execution engine, making the creation of computational
pipelines, and running them across a compute cluster, easy to do.  The software takes a templated
RMS script plus spreadsheet data files, generates commands by using the spreadsheet data to fill
in the templates (i.e., for each file, each sample, each trio), and runs them on the current
computer or across a cluster.

And, calling it a language overstates the case just a bit.  It really consists of a little extra
syntax to organize the Bash, Perl, Python or R scripts of a pipeline, along with fill-in-the-blank
“template elements” that are replaced when creating the executed commands.  You write the pipeline
steps in any combination of the four languages (whatever best fits each step’s implementation),
and the syntax is designed so that language-specific editors will ignore the RMS elements of the
scripts, allowing you to use your favorite editor tools to do your pipeline development.


Contents:

.. toctree::
   :maxdepth: 2



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

