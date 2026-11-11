### Objectives:

> The primary objective is to conduct a thorough investigation into a suspected Remote File Inclusion Incident with the goal of identifying the source, scope, and impact of the attack

### Incident Details: 

> Investigate the suspicious RFI activity detected by the security monitoring systems. Investigate the web logs to identify the targeted IP, source, and potential impact.

## ==🔴 Targeted URL: ==

### http://192.168.117.190/prod/vulnerabilities/upload/


### About RFI: 

> RFI is a type of web-based security vulnerability exploit that occurs when an attacker is able to upload suspected files including php, ps1, exe, ddl etc directly into the web server. This can happen when the upload function of a web application has not been properly sanitised, by exploiting such a vulnerability the attacker can directly deploy the payload and gain a stable shell with the targeted web server.

![[Pasted image 20261110142739.png]]

### Investigation MindMap:

![[Pasted image 20261110142901.png]]

### Steps :

#### 1. in Wazuh will use index : wazuh–archives

#### 2. Retrieving the **web-accesslog** : decoder 


>  Decoders are the key components which are responsible for parsing raw logs into a structured format that can be easily analzsed and understood by the system.

```
decoder.name : web-accesslog
```

![[Pasted image 20261110000456.png]]
#### 3. Examine the access logs 

> In general web server logs contain valuable information about incoming requests, errors, and server activities. It is recommended to check the access logs which record every request made to the server.


```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log"
```

![[Pasted image 20261110000556.png]]

### a. Examine the URI 

### After Checking I found in data.url :  fi*

```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url:  /prod/vulnerabilities/f*
```

![[Pasted image 20261110145019.png]]
## ==🔴 Note : at /prod/vulnerabilities/f* we have 6 , include password , php   ==


### After Checking I found in data.url :  up*

```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url:  /prod/vulnerabilities/up*
```

![[Pasted image 20261110144727.png]]

## ==🔴 Note : at /prod/vulnerabilities/up* we have 2 ==

### b. Investigate the POST request  

### we found in the protocol : Get & Post when upload will use POST

```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url:  /prod/vulnerabilities/up* and data.protocol: *

```

![[Pasted image 20261110145214.png]]


```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url:  /prod/vulnerabilities/up* and data.protocol: POST

```

![[Pasted image 20261110145337.png]]


### c. Co-relate with PCAP

> In-general POST request doesn’t provide a high visibility, in such it is recommend to correlate with network traffic, in our scenarios we have attached the PCAP file of the specific activity named: suspected.pcapng


![[Pasted image 20261110155942.png]]

## IP Attacker : ==🟡192.168.117.143 ==
### ==🔴 Note : For the above result we identified that the src has been try access the url /prod/hackable/uploads/rev[.]php==

### d. Investigate the detected file activity 

> investigate into file that we get /prod/hackable/uploads/rev.php

```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url: "/prod/hackable/uploads/rev.php"
```


![[Pasted image 20261110160551.png]]


### e. Identify the root cause

> After gathering every possible information it's time to determine the root cause for the detected activity, typical RFI will maintain a stable shell with the attacker machine. To determine the particular activity again redirect to the PCAP file and execute the following command

![[Pasted image 20261110160752.png]]

![[Pasted image 20261110153558.png]]

## File found : ==🔴rev.php== 
## IP Attacker : ==🟡192.168.117.190 ==

> In-real case scenarios the network traffic might be too hard to be prioritized due to its large network connectivity, in such cases it's recommended to check based on the suspected port based access. As per the observation over the PCAP we observed that the Src is been trying to communicating to an suspected IP over the port 9001

![[Pasted image 20261110162714.png]]

```
ip.dst == 192.168.117.190 && tcp.srcport ==9001
```

