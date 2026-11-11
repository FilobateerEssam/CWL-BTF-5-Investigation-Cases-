
### Objectives:

> The primary objective is to conduct a through investigation into a suspected SQL-injection Incident with the goal of identifying the source, scope, and impact of the attack. 


### Incident Details: 

> Investigate the suspicious SQL-Injection activity detected by the security monitoring systems. Investigate the web logs to identify the targeted IP, source, and potential impact.

## ==🔴 Targeted URL: ==

### http://192.168.117.190/prod/vulnerabilities/sqli

### About SQLI: 

**SQL injection** is a type of web-based security vulnerability that occurs when an attacker is able to manipulate or inject malicious SQL code into a query. This can happen when input from an application, such as a web form or URL parameter.

![[Pasted image 20261109223304.png]]

## Investigation MindMap:

![[Pasted image 20261109223406.png]]

### Steps :

#### 1. in Wazuh will use index : wazuh–archives

#### 2. Retrieving the **web-accesslog** : decoder 

>  Decoders are the key components which are responsible for parsing raw logs into a structured format that can be easily analzsed and understood by the system.

![[Pasted image 20261109233635.png]]

#### 3. Examine the access logs 

> In general web server logs contain valuable information about incoming requests, errors, and server activities. It is recommended to check the access logs which record every request made to the server.

```
location : "/var/log/apache2/access.log"
```

### ==🔴Note: Access logs provide details like IP addresses, requested files, status codes, etc.==

#### 4. Traffic Pattern Analysis 
	a. Examine the URI 
	b. Observe the Status Codes 
	c. Identify IP associated with the activity 

### a. Examine the URI 
```
location : "/var/log/apache2/access.log" and  data.url: /prod/vulnerabilities/sqli*

```

#### 💹 Visualization

![[Pasted image 20261109234414.png]]

![[Pasted image 20261111140209.png]]
#### b. Observe the Status Codes 

```
location : "/var/log/apache2/access.log" and  data.url: /prod/vulnerabilities/sqli* and data.id: *

```

![[Pasted image 20261109235025.png]]

#### ==🔴Note: HTTP 200 generally indicates a successful HTTP request. This particular request appears when a client sends a GET request to the server, and the server successfully retrieves the requested resource, it responds with a status code of 200.==

### C. Finding the IP 

![[Pasted image 20261109235232.png]]

## IP Attacker : ==🟡192.168.117.143 ==



