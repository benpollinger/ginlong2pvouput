
# ginlong2pvoutput support scripts
These bash scripts support the running of the python scripts to gather, process and upload data. Configure cron jobs to call some of them (see below) - others are called in turn.

I run this on a Raspberry Pi running [OpenMediaVault](https://www.openmediavault.org/) on Raspbian, and found not all scripts work as I'd hope when placed in my home directory. I think that is to do with the noexec flag. Hence I moved everything to `/usr/local/src/ginlong2pvoutput` and all is fine.

Configure your `scriptdir=` lines according to your system.


## runginservpoller

Start this script at boot time (see `crontab` below). Runs the poller (the `ginserv.py` Python 3 program) in a loop.
- It changes directory to the one with this code and the log files.
- If the poller is already running when the script starts, it kills it and waits 5 minutes.
- It renames any ginserv.log file to ginserv.log\_YYYYMMDD\_HHMMSS.
- It then starts the poller, logging to a ginserv.log\_YYYYMMDD\_HHMMS file.
- When that poller dies, it waits 5 minutes and loops round again.

## killginservpoller

This looks for the ginserv.py process and kills it. Called from various places.

## killiflogempty

This script is used to kill off a poller program if it seems to be stuck in the startup sequence (more of a problem with the TCP-based poller, but does sometimes happen with the UDP one).

It looks for the most recent ginserv.log\* file. if it only contains the startup sequence, the `killginservpoller` script is called. The `runginservpoller` loop will restart it 5 minutes later.

## midnightcheck_ginservpoller

This is run just before midnight (when, in much of the world, there is little sunlight!). The 5-minute delay in runginservpoller means that the next run starts the next day.

It kills the currently-running poller, concatenates all of the log files for today into `logs/$today.ginserv.log`, then calls the `ginserv2pvoutput` script to process that file and upload data to pvoutput.org

When testing out your setup, you can call this earlier in the evening using an amended crontab entry. For example, my inverter sleeps once it's dark at about 7pm now, so I run it just after that.

# Crontab entries

Change the target directory to wherever you settled on. They should work fine run as normal user (don't need root).

    # Start the ginservpoller at boot time
    @reboot /usr/local/src/ginlong2pvoutput/runginservpoller

    # Stop the ginserv poller just before midnight, and check that some entries were written today
    57 23 * * * /usr/local/src/ginlong2pvoutput/midnightcheck_ginservpoller

    # Kill the ginserv poller if it hasn't written data by lunchtime
    2 10,11,12,13 * * * /usr/local/src/ginlong2pvoutput/killiflogempty
