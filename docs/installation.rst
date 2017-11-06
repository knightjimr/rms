
Installation and Configuration
==============================

Installation
------------

The software is available at https://github.com/knightjimr/rms, and the latest stable release is
available at https://github.com/knightjimr/rms/releases.  No pip installation script is available,
yet, so the easiest way to install is to download the release, untar it and then add the directory
to your PATH environment variable.  The software is a pure python implementation, with no
dependencies.  It was developed using python 2.7, but should be compatible with 2.6+ and 3.*
versions.

Configuration
-------------

Configuring RMS to use your cluster takes a little more work, because every cluster's job scheduler
has a different setup, and many of the resource limit rules are not detectable automatically.  So,
the first thing you need to do is learn what job queues you can submit jobs to, and what, if any,
resource limits there are.  Hopefully, if you've been using the cluster for a while, you should know
much of this information.

Another helpful thing to do with some clusters may be to test out the resource limit options using
interactive jobs, to see what options ensure that you get compute node resources quickly.  The
documentation for one cluster I tried out said they had 20 core compute nodes, and said they kill
jobs after 24 hours.  When I started trying those options with interactive jobs, I found setting the
time limit to 24 hours was fine, but trying ask for 20 cores on a single host caused the job to just
sit in the queue.  When I changed it to 10 cores, I was able to get my jobs running quickly.
Setting your resource limits so that jobs start quickly will allow RMS to start on your computations
quickly, and then be able to ramp up quickly when the cluster is not busy.

Defining the Cluster
^^^^^^^^^^^^^^^^^^^^

RMS first reads the file "RMSRC" in the same directory as the RMS software, to get the system-level
configuration, then tries to read "~/.rmsrc" for any user changes to the configuration.  Similar to
ulimit, user changes only reduce the resource limits defined in the system RMSRC, although there is
an override option to allow a user to change the limits, regardless of what the system configuration
is (at their own risk...since, if the resource limits don't match the actual cluster configuration,
their jobs will just sit in the queue forever...).

The RMSRC (or ~/.rmsrc) lines have the format "queueName option=value..." to define the job queues.
For example, the first cluster I used was a PBS/Torque cluster with three job queues, "default",
"highcore" and "bigmem", each with different configurations and where highcore and bigmem were to be
used only when needed.  So, the RMSRC file for that cluster was the following: ::

   default type=torque;cpulimit=100;ppn=8;default=true
   bigmem type=torque;joblimit=4;ppn=64
   highcore type=torque;ppn=48

All were of type "torque", the default queue had a limit of 100 cores and had compute nodes with 8
cores each.  The bigmem queue only allowed 4 jobs at a time and each compute node had 64 cores.  The
highcore node had no resource limits, and had 48 cores per node.

The current cluster is a Slurm cluster, with "general" and "scavenge" queues, each with a 300 core
limit, with compute nodes that have 20 cores and 124GB of memory (where Slurm memory management is
turned on, so you must request and stay within the memory limit or the job is killed), and a one
week job time limit.  The RMSRC file for that is the following: ::

   general type=slurm;ppn=20;mem=124;cpulimit=300;walltime=168;wallbuffer=12;default=true
   scavenge type=slurm;ppn=20;mem=124;cpulimit=300;walltime=168;wallbuffer=12

The options string for each queue line contains a semi-colon separated list of "name=value" pairs,
defining the properties of that queue.  The core properties that must be defined for each queue
are the following:

   type
      This defines the type of the job scheduler (see below for the list of supported job schedulers
      and their values).
   ppn
      This defines how many cores to request when submitting an RMS worker to run on a compute node.
      RMS creates one long-lived worker for each compute node it uses, and passes multiple commands
      to that worker, so the ppn value should be the number of cores that the compute nodes in this
      job queue have, not how many might be used in the RMS script steps.  However, you can set this
      to request only partial access to the nodes (i.e., 10 out of 20 cores), and RMS will only use
      that proportion of the cores, memory and tmp space on the compute nodes.
   default
      This says whether to use this queue by default, if the list of queues is not explicitly named
      in the "-n" command-line option.  [default:  false]
   
There are a number of properties that can be used to define the resource limits for the queue.  RMS
interacts with the job queue by submitting one job for one compute node, in order to run the RMS
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
   mem
      This defines the memory limit that a job can use.  This memory limit will be passed as an
      option to the job scheduler (and be used as the limit for what can be run by the job).
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
   account
      This defines the account to be passed as an option to the job scheduler.
   queue
      This defaults the actual cluster queue to be used to submit jobs, for this RMS "queue".
      The use of this argument allows you to create multiple RMS "queues" for a single cluster
      queue, in order to either use different numbers of cores or memory (if your cluster
      has different kinds of compute nodes on the same queue) or use different accounts (if
      your cluster has account processing and you have multiple accounts that can be used).
      The value of this option can also be a previously defined RMS "queue", and will copy
      the options set for that "queue" as the defaults for this "queue".

Supported Job Schedulers
^^^^^^^^^^^^^^^^^^^^^^^^

The software is setup to handle a number of different job schedulers, but not all are supported
(because I don't have access to clusters with other schedulers to test the functionality).  The
list of supported clusters is the following (listed by the value to use for the "type" property
above):

   torque
      The PBS-Torque job scheduler.
   lsf
      The Platform LSF job scheduler.
   slurm
      The SLURM job scheduler.

The list of schedulers that the software is ready to support, but has not been tested, is the
following:

   pbs
      The PBS job scheduler.
   sge
      The SunGrid Engine job scheduler.

If you are willing to help test one of these schedulers, or have a different job scheduler on
your cluster, please contact knightjimr@gmail.com.

Aliases
^^^^^^^

Because of some of the complexities of interactive vs. non-interactive bash shells, any aliases that
you've defined in your ~/.bashrc file cannot be used in RMS scripts (trying to work around that
actually caused more initialization errors for more peoples' scripts).  RMS automatically sets the
"expand_aliases" option at the beginning of every bash script it runs, so even if your version of
bash disables aliases by default, they can be used anywhere in the bash sections of RMS scripts.
However, trying to use an alias defined in the ~/.bashrc file will not.

To support those aliases, RMS has adopted the "standard" workaround that other software uses.  RMS
looks for and loads a file ~/.alias at the beginning of each shell script, if that file exists.  So,
if you have defined aliases in your ~/.bash_profile or ~/.bashrc file that you would like to use in
RMS scripts, copy those aliases into ~/.alias, and then add the following lines to your ~/.bashrc
script: ::

   if [ -f ~/.alias ]; then
      . ~/.alias
   fi

(and possibly your ~/.bash_profile, if that script does not have the standard lines which load your
~/.bashrc file every time it runs.)

Note that some bash shell have alias expansion turned on by default, in which case this may not be
necessary (I don't currently have access to such a machine, so I have not tested it).
