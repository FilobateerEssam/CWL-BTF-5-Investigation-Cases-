### Objectives:

> The primary objective is to conduct a thorough investigation into a suspected Cross Site Scripting Incident with the goal of identifying the source, scope, and impact of the attack


### Incident Details: 

> Investigate the suspicious Cross Site Scripting activity detected by the security monitoring systems. Investigate the web logs to identify the targeted IP, source, and potential impact.

## ==🔴 Targeted URL: ==

### http://192.168.117.190/prod/vulnerabilities/xss_r/ 

### About XSS: 

XSS is a type of web-based security vulnerability that occurs when an attacker is able to manipulate or inject malicious Java Script commands into a query. This can happen when input from an application, such as a web form or URL parameter.

![[Pasted image 20261109235950.png]]

## Investigation MindMap:

![[Pasted image 20261110000054.png]]

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
### After Checking I found in data.url :  XSS

```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url: /prod/vulnerabilities/*
```

![[Pasted image 20261110000703.png]]

## ==🔴 Note : at /prod/vulnerabilities/* we have 19 ==

```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url: /prod/vulnerabilities/xss*
```

## ==🔴 Note : at /prod/vulnerabilities/xss* we have 6 ==

![[Pasted image 20261110001014.png]]

> As per the above result, we observed multiple XSS commands over the web url request. In general, it's not considered as a normal activity when we observe a Java Script command over the URL/URI requests

#### 💹 Visualization

![[Pasted image 20261110001638.png]]

#### b. Observe the Status Codes 

```
decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url: /prod/vulnerabilities/xss* and data.id: *
```

![[Pasted image 20261110001746.png]]

#### As per the observation we found multiple requests with a status code 200, 404

#### ==🔴Note: HTTP 200 generally indicates a successful HTTP request. This particular request appears when a client sends a GET request to the server, and the server successfully retrieves the requested resource, it responds with a status code of 200.==

### C. Finding the IP 

![[Pasted image 20261110002036.png]]

## IP Attacker : ==🟡192.168.117.143 ==
