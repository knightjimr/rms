
RMS Execution Process
=====================

When RMS runs, it performs the following operations (understanding this can help with writing RMS scripts).

  * Reading and parsing the RMS script
  * Processing the RMS command options
  * Constructing the spreadsheet data
  * Executing the step commands
      - Determining commands and dependencies
      - Writing a command's script
      - Starting and Ending Worker Jobs
      - <local> and <tmp> directories
      - Running a command
      - Collecting stdout and stderr
      - Reporting progress

Troubleshooting tips.
      

