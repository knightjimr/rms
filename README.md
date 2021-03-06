RMS - Run My Samples
====================

RMS is a tool for writing/running computational pipelines on a computer or across a cluster.  It is
more of a "cluster scripting language" than a workflow language like CWL.  The bulk of the script is
the bash, Python, R or Perl code that make up the steps of the pipeline, plus some extra lines to
organize those steps and fill-in-the-blank "template elements" that are replaced when the script is
used to generate commands.

Documentation for the tool can be found at http://rms.readthedocs.io.

The software is a pure python implementation, with no dependencies.  It was developed
using python 2.7, but should be compatible with 2.6+ and 3.* versions.

After installation, configuring and testing with your cluster is a little more work.  Follow the
instructions in http://rms.readthedocs.io/en/latest/installation.html to setup the configuration for
your cluster, and run the tests in http://rms.readthedocs.io/en/latest/hello1.html to make sure it
is working.

