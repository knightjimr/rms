
Installation and Configuration
==============================

Installation
------------

[Note:  The RMS software is not currently available on github, but will be once the documentation is completed.]

RMS can be installed using the following command: ::

   pip install git+https://www.github.com/knightjimr/rms

or by simply cloning the package from github ::

   git clone https://www.github.com/knightjimr/rms.git

and adding the "rms" directory to your $PATH.  The software is a python implementation, with no dependencies.
It was developed using python 2.7, but should be compatible with 2.6+ and 3.* versions.

Configuration
-------------

Configuring RMS to use your cluster takes a little more work, because every cluster's job scheduler has a
different setup, and many of the resource limit rules are not detectable automatically.  So, the first thing
you need to do is learn what job queues you can submit jobs to, and what, if any, resource limits there are.
Hopefully, if you've been using the cluster for a while, you should know much of this information.

The way to tell RMS about the cluster is through an ".rmsrc" file.  RMS will look first at the "RMSRC"
environment variable, then look for the file "$HOME/.rmsrc" in your home directory.  If it does not find
either, it assumes that the cluster has a PBS-Torque scheduler, with a "default" queue that can be used
to submit jobs.

The .rmsrc file contains "name=value" lines that let you define the job queues, as well as some default option
values for RMS.  The supported lines are the following:

   mode=value
      This sets the default RMS running mode to "value", where "value" can be "test", "single",
      "parallel" or "cluster".  So, if you don't have a cluster, and just want to use RMS on
      your current computer, add "mode=parallel" to the .rmsrc file.
   queue=name options
      This defines a job queue named "name", with one or more properties given in the 
      "options" string (see below for the format of the options string).

For example, the cluster that I use has four job queues, "default", "highcore", "bigmem" and "state", each with
different configurations, and for my general work I should use the default queue, but can use the highcore and
bigmem when necessary, and have the ability to use the state queue if I ask permission first.  So, my .rmsrc
file looks like this: ::

   queue=default use=true;type=torque;cpulimit=100;ppn=8
   queue=state use=false;type=torque;ppn=16
   queue=bigmem use=false;type=torque;joblimit=4;ppn=64
   queue=highcore use=false;type=torque;ppn=48

The options string for each queue line contains a semi-colon separated list of "name=value" pairs, defining
the properties of that queue.  The core properties that should be defined for each queue are the following:

   type
      This defines the type of the job scheduler (see below for the list of supported job schedulers
      and their values).  [default:  torque]
   ppn
      This defines how many cores to request when submitting an RMS worker to run on a compute node.
      RMS creates one long-lived worker for each compute node it uses, and passes multiple commands
      to that worker, so the ppn value should be the number of cores that the compute nodes in this
      job queue have, not how many might be used in the RMS script steps.
   use
      This says whether to use this queue by default, if the list of queues is not defined by the
      "-n" command-line option.  [default:  true]
   
In addition, there are a number of properties that can be used to define the resource limits for the
queue.  RMS interacts with the job queue by submitting one job for one compute node, in order to run the RMS
worker process on that node (and will do that multiple times, as needed, so that RMS can expand and
contract the number of compute nodes used, based on the commands ready to be run in the script).

   cpulimit
      This defines the limit of cores that this queue will allocate to a user.  RMS will
      stop submitting jobs for RMS workers when it reaches this limit.
   joblimit
      This defines the limit on the number of jobs that a user can submit to the queue.  RMS
      will only submit at most this number of RMS worker jobs.
   nodelimit
      This defines the limit on the number of nodes that a user can submit to the queue.  RMS
      will only request at most this number of compute nodes.
   walltime
      This defines the time limit that a job can run, in hours.  This time limit will be passed
      as an option to the job scheduler.
   wallbuffer
      This defines the buffer time, in hours, that RMS should use for any job where walltime
      is defined, in order to stop sending commands to the worker that is near being killed.
      To avoid interrupted commands, this should be set longer than any command
      may run.  For example, if a queue defines "walltime=24;wallbuffer=2", RMS will submit a
      worker job with a time limit of 24 hours, send commands to that worker for the first 22
      hours, then stop sending commands at the 22 hour mark, and wait for that worker to die
      as soon as all of the executing commands are completed, or the worker is killed.
      (If more commands need to be run, new worker jobs will be submitted.)
   tmp
      This defines the location of the local tmp space on a compute node.  [default:  /tmp]

Supported Clusters
------------------

The software is setup to handle a number of different job schedulers, but not all are supported
(because I don't have access to clusters with other schedulers to test the functionality).  The
list of supported clusters is the following (listed by the value to use for the "type" property
above):

   torque
      The PBS-Torque job scheduler, which is the default.
   lsf
      The Platform LSF job scheduler.

The list of schedulers that the software is ready to support, but has not been tested, is the
following:

   pbs
      The PBS job scheduler.
   slurm
      The SLURM job scheduler.
   sge
      The SunGrid Engine job scheduler.

If you are willing to help test one of these schedulers, or have a different job scheduler on
your cluster, please contact knightjimr@gmail.com.
