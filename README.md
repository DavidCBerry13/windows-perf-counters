# Windows Performance Counters

This repo contains a collection of scripts for working with Windows Performance Counters and troubleshooting performance issues.

## List all of the Performance Counters on a computer

Use the [typeperf](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/typeperf) command with the `-q` option to list all pf the performance counters on a machine.

```cmd
typeperf -q
```

It is usually easier to write all of the counters to a file so you can view/edit them with a text editor.  This can be easily done by using the `-o` flag and specifying an output file.

```cmd
typeperf -q -o counter-list.txt
```

## Create a Data Collector Set with logman.exe

### Basic Example

You could manually go into perfmon and create data collector sets, but it is easier and more repeatable to use the [logman](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/logman-create) command line utility.

The following comamnd creates a data collector set named `Standard-Perf-Data`.

```cmd
logman.exe create counter 
    - n Standard-Perf-Data ^
    -f bincirc ^
    -o c:\perflogs\Standard-Perf-Data
    -v mmddhhmm ^
    -max 250 ^
    -si 00:05:00 ^
    -c "\LogicalDisk(*)\*" "\Memory\*" "\Network Interface(*)\*" "\Paging File(*)\*" "\PhysicalDisk(*)\*" "\Process(*)\*" "\Redirector\*" "\Server\*" "\System\*"   
```

- The `-n` option is used to specify the name of the data collector set
- The `-f` option is used to specify the type of output file.  `bincirc` is a circular binary file that will write over itself when a certain size is reached.  Other options are `bin` (binary), `csv` (CSV), `tsv` (Tab separated), and `sql` (write to a SQL Server)
- The `-o` option specifies the output location of the data file.  Binary files will have a `.BLG` extension.
- The v option specifies the versioning info into the filename
= The `max` option is the maximum file size for the datafile (useful to make sure you don't fill up your disk)
- The `si` option specifies the sample interval in `hh:mm:ss` format
- The `c` option takes a list of counters to include as part of the data collector set

### Specify Perf Counters in a File

It is often easier to specify the perf counters you want to capture in a text file rather than typing all of them on the command line.  This is done with the `-cf` option.  The file should contain one performance counter per line.

```cmd
logman.exe create counter 
    - n Standard-Perf-Data ^
    -f bincirc ^
    -o c:\perflogs\Standard-Perf-Data
    -v mmddhhmm ^
    -max 250 ^
    -si 00:05:00 ^
    -cf standard-counters.txt   
```

### Scheduling Perf Counters to be Collected for a specified interval

You can also specify when you want data collection to start and end with the `-b` and `-e` options.

```cmd
logman.exe create counter 
    - n Standard-Perf-Data ^
    -f bincirc ^
    -o c:\perflogs\Standard-Perf-Data
    -v mmddhhmm ^
    -max 250 ^
    -si 00:01:00 ^
    -b <M/d/yyyy h:mm:ss[AM|PM]>
    -e <M/d/yyyy h:mm:ss[AM|PM]>
    -cf standard-counters.txt   
```

## Converting a binary log file to CSV

It is useful to collect the performance counters into a binary circular file as this is the most efficient format and the file can roll over itself.  But to bring into Excel, you want a CSV file.  Use the relog command to convert a binary (.blg) file to a CSV file as follows.

```cmd
relog <binary-file-name> /o <csv-file-name>
```


## Additional Resources

- [**Capture Performance Logs Using Logman.exe, relog.exe and typeperf.exe**](https://www.youtube.com/watch?v=VNcbOfZWvAA) - YouTube Video by Dell Enterprise Support showing the basics of setting up Performance Counters on a Machine
- [**Two Minute Drill: LOGMAN.EXE**](https://techcommunity.microsoft.com/t5/ask-the-performance-team/two-minute-drill-logman-exe/ba-p/373061) - Blog post quickly summarizing the use of logman to create data collector sets
- [**Hyper-V - Detecting Virtualized Environment Bottlenecks**](https://learn.microsoft.com/en-us/windows-server/administration/performance-tuning/role/hyper-v-server/detecting-virtualized-environment-bottlenecks)