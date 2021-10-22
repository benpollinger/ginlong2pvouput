# WORK IN PROGRESS

# ginlong2pvouput

This builds on https://github.com/simon3270/ginlong-python for automatically capturing the data from a Solis / Ginlong solar inverter (mine is a Solis 3.6-DT-DC), processing them and uploading automatically to pvoutput.org at the end of each day.

The `ginserv.py` and `ginserv_tcp.py` scripts are unchanged - they capture the data from the inverter but don't process it further. `get_data.py` has been changed slightly to make further processing simpler using the Bash scripts I've written. These uploading each day's data automatically to pvouput.org, using their API.

## ginserv2pvoutput

Runs in the evening (set up with cron). This is the main script which calls a few others:
1. Calls `get_data.py` to convert today's `ginserv.log` to `pvoutput.log`
2. Checks if the day's log contains less than 50 records, which likely indicates a problem with the inverter sending data or the ginserv.py script not catching it properly. You can configure it to email you or use [IFTTT](https://ifttt.com/) to notify you.
3. Splits the day's log into 30 line chunks - this is the limit of one call to the pvoutput.org `addbatchstatus.jsp` API service
4. Parses each 30 line chunk into a string which is sent to the pvoutput.org API using curl
5. The last few commands clean up temporary files and backs up the logs.

There are 3 configuration fields:
1. `pvoutput_api` is your API key from https://pvoutput.org/account.jsp
2. `pvoutput_site` is your system id number from pvoutput.org - at the bottom of https://pvoutput.org/account.jsp
3. `api_url` is entered already, can be changed here if pvoutput.org ever changes it.

## dailytotals

Steps through each daily log (in pvoutput format) in the logs folder, creates a monthly summary of each day's kWh total, time of first and last entry, and number of lines.

## showlatest

Finds the most recent pvoutput.log in the logs folder and shows it using cat

## uploadpvo

Uploads a specified pvoutput.log file to pvoutput.org - call it with uploadpvo logs/logfilename





The `support` directory contains scripts to support running of the above logging programs, and `crontab` entries to run the scripts.
