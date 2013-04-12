
# What is the log formatter and how to use it?
  This page describes what the log formatter is and its applications.

## What is this?
   The log formatter is a mechanism of the logger subsystem of libsheepdog. It can be used for choosing log fomrat. Currently, 2 formats are supported: default and JSON.

## How to choose the format
   Current users of the formatter are sheep and shepherd. Their common command line option -F is used for log format choose. If you pass them an option "-F default", it will do nothing. The format of logs produced by the programs is same to the format when you don't pass "-F default". You can pass them "-F json" as an alternative. If you pass the option, the programs produce logs formatted in JSON.

## What is the purpose of formatting logs in JSON?
   The logs formatted in JSON are (a little bit) structured. This is a typical example of the logs:
<pre>
{ "user_info": {"program_name": "sheep", "port": 7000},"body": {"second": 1365731341, "usecond": 742602, "worker_name": "main", "worker_idx": 0, "func": "init_signal", "line": 173, "msg": "register signal_handler for 10"} }
</pre>
   As this form implies, we can extract information like timestamps, function names, lines, etc without adhoc matching of regular expression.

## Application of the JSON format
   Currently, there is an only one application of the JSON log format: script/json_log_viewer.py. This script is for viewing JSON logs produced by the tests of sheepdog. It takes file names of JSON logs as its arguments:
<pre>
$ script/json_log_viewer.py "json log file 1" "json log file 2" ...
</pre>
   You can view pretty-printed logs with this script. In the result, logs have different colors based on their producers and timestamp are showed as a delta from the timestamp of the first log.

### Key bindings of json_log_viewer.py
* j: scroll down one line
* k: scroll up one line
* g: go to top
* G: go to bottom
* q: quit

### Typical example of json_log_viewer.py with tests
<pre>
$ cd tests
$ sudo LOG_FORMAT=json DRIVER=local ./check 001 # specify log format with the environment variable $LOG_FORMAT.
$ ../script/json_log_viewer.py logs.timestamp/* # view the collected log files
</pre>

## Future work
* Improve functionality of json_log_viewer.py
* State machine based analysis of sheep. We can analyze various behavior of sheep with JSON logs and state machine based analyzer (e.g. memory consumption, I/O).
