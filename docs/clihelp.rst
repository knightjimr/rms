
Command-Line Help Text
======================

Usage: rmsScript [conditional processing]
       rms [options] rmsScript [conditional processing]

   RMS executes pipeline scripts on the current machine or across a cluster.  Similar to perl
   and python, the script can be run either as "rms myscript ..." or just "myscript" (if the
   first line of the script file is "#!/usr/bin/env rms").

   The command line arguments after the script file are "conditionally processed", meaning that if
   the script defines command line processing, the arguments are processed by the script.  If the
   script does not define command line processing, the following default command line processing
   occurs:

       myscript [options] sheet...
       rms myscript [options] sheet...

    where "sheet" is one or more tab-, comma- or space-delimited spreadsheet files (see the online
    help files for more details), and the following options are supported.

       * -t or --test:  Test the script for syntax errors (by compiling only)
       * -s or --single:   Run the script sequentially on the current computer
       * -p or --parallel:  Execute the script just on the current computer (like GNU parallel)
       * -c or --cluster:  Execute the script across the cluster
              [default mode:  cluster]

       * -S step  or --start step:  Start the pipeline with step "step" (skipping initial steps)
       * -E step  or --end step:    End the pipeline with step "step" (skipping later steps)
       * -O step  or --only step:   Only run step "step"  (equivalent to "-S step -E step")

       * -f  or --force:  Ignore existing files and force the pipeline commands to run

       * -o dir  or --output=dir:  Set the output directory (and current working directory for the
                                   script) to "dir".    [default:  .]
       * -l prefix or --log=prefix:  Log the script execution stdout and stderr to "prefix.stdout" and
                                     "prefix.stderr".  Passing "-" outputs the execution stdout/stderr
                                     to the command's stdout/stderr.
                                 [default:  RMS_myscript_YMD_HMS]

       * -n #  or  --num=#:  Limit for the number of nodes to use (cluster mode) or the number of
                             cores to use (parallel mode)
       * -n queue[:#],...  or --num=queue[:#],...:
                             The queues to use and, optionally, node limits for each queue
                             (cluster mode only, 0 specifies no limit)
              [parallel default:  Number of cores of computer]
              [cluster default:  "default" queue, no limit]

    When the command-line explicitly begins with "rms", as in "rms [options] myscript ...", all
    of the above options may be specified between "rms" and "myscript" (so that you can specify the
    runtime execution of the program, even if the script redefines its command-line processing).
