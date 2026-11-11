
### Objectives:

> The primary objective is to conduct a through investigation into a suspected memory dump with the goal of identifying the source, scope, and impact of the attack

### Incident Details: 

> Investigate the suspicious endpoint compromise activity by the Endpoint monitoring systems. Investigate the .RAW file to identify the targeted IP, source, and the root cause of the activity

### ==🔴 Memory Dump : suspected.raw  =

### About Memory Dump Analysis : 

> Memory dump analysis plays a critical aspect of identifying the root cause and incident response, A memory dump, also known as a core dump or a crash dump, is a file that captures the contents of a computer's volatile memory at a specific point in time. This snapshot of memory can provide valuable information for understanding system behaviour, identifying anomalies, and investigating security incidents

![[Pasted image 20261111121628.png]]

### Investigation MindMap:

![[Pasted image 20261111121700.png]]

## Steps :

#### 1. Download the given memory dump on your own host machine in an isolated environment.

![[Pasted image 20261111124605.png]]

#### 2 .investigation we will be processed using the volatility tool

> Investigating memory dump involves careful steps to ensure security and determine the nature and potential threats associated with the web logs. 
> 

#### 3. Gather the information 

> The first step of investigation begins by analyzing the basic information such as the operating system version, architecture, and system configuration can aid in accurately identifying the system being analyzed.

```
sudo python3 vol.py -f <file_path> windows.info.Info
```

<file_path> : file of suspected.raw

![[Pasted image 20261111130051.png]]
#### 4. Examine the running process 

> After gathering the basic information the next step is to investigate and begin by analysing the behaviour of running processes, this particular approach will help in understanding normal and abnormal activities on the detected compromised system.

```
sudo python3 vol.py -f <file_path> windows.pslist.PsList
```

> Searching for the suspected process in a memory dump is like trying to find a needle in a haystack. It generally requires a high level of skill-set of understanding, it's recommended to go through each and every running process and filter-out the suspicious process.


![[Pasted image 20261111130523.png]]
![[Pasted image 20261111131029.png]]

#### as you see the ==🟡shell.exe==  is  ==🔴true== 

#### 5. Determine the command line activity 

> After determine the suspicious process its recommend to cross verify the process with the command line activity execute the below mentioned commands and observe the findings

```
sudo python3 vol.py -f <file_path> windows.cmdline.CmdLine

```

![[Pasted image 20261111132807.png]]

As per the above result we did not observe any suspicious command execution associated with the executables

> Generally many malicious exe are been associated with multiple command execution activity, it's been recommended to cross-verify the activity which are associated with the detected exe

#### 6. Download the detected DUMP file 

> Next step of the investigation is to download the detected suspicious file for further investigation to, using our raw image file we can easily retrieve the file using the PID value associate with it, execute the below mentioned commands and observed the result

```
sudo python3 vol.py -f <file_path> windows.pslist --pid 7396 --dump

```

![[Pasted image 20261111133655.png]]

#### After extracting the dump the next step is to check for the strings associated with the dump file

```
sudo strings pid.7396.0x400000.dmp 
```

![[Pasted image 20261111134202.png]]

> In majority of the cases we might find interesting value but we did not observed any intresting value in our case it recommended to analzse the dump via virustotal, upload the extracted dump directly into the virustotal for further analysis

![[Pasted image 20261111134321.png]]

#### 7. Examine the network traffic

```
sudo python3 vol.py -f <file_path> windows.netstat.NetStat
```

![[Pasted image 20261111135004.png]]

### As per the above result we clearly observed that the detected executable ==🟠shell[.]exe== has been successfully communicating to an ==🟡IP 192[.]168[.]121[.]242== via ==🔴port 4444==


