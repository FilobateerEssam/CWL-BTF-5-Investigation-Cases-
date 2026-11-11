
### Objectives:

> The primary objective is to conduct a thorough investigation into a suspected external network communication Incident with the goal of identifying the source, scope, and impact of the attack

### Incident Details: 

> Investigate the suspicious external communication activity detected by the network monitoring systems. Investigate the PCAP to identify the targeted IP, source, and potential impact

## PCAPFile Name: ==🟡 external.pcapng ==

## ==🔴 Targeted Port:  4444 ==


### About RFI: 

> External network communication refers to the interaction and data exchange between a local network and external networks, when a attacker successfully deploy the payload over the host machine it will establish a stable connection between the attacker and the victim 

![[Pasted image 20261110164646.png]]

### Investigation MindMap:

![[Pasted image 20261110164713.png]]

> Wireshark is a popular open-source network protocol analyzer that allows users to capture and inspect the data travelling back and forth on a network in real time
## Steps :

#### 1. Prioritize the detected port activity

#### ==🟡 Statistics → Endpoints → TCP or UDP ==

```
tcp.dstport == 4444
```

#### 2. Identifies source and destination IP addresses

> The next step is to identify and retrieve the IP activity which are associated with the detected port activity, from the above result we identified that the src 192.168.132.238 → 192.168.132.242 over destination port 4444
### List All IP Addresses

#### ==🟡 Statistics → Endpoints ==

### List All Conversations (IPs + Ports)

#### ==🟡 Statistics → Conversations ==

You'll see:

- Source IP
- Destination IP
- Source Port
- Destination Port
- Packet and byte counts

```
tcp.dstport == 4444 && ip.src == 192.168.132.238 && ip.dst ==
192.168.132.242
```

![[Pasted image 20261110170624.png]]
#### 3. Examine the traffic flow

> To examine the network traffic it is recommended to check the packet flag from the above observation result. We identified SYN ACK this pattern observed when a src dest established a stable communication, and 

### ==🔴PSH== command generally observed when a Src trying to transmit data or executing command.

![[Pasted image 20261110170940.png]]

> This particular pattern was observed when a victim got compromised and established a stable shell with the attacker machine.

![[Pasted image 20261110171121.png]]

### ==🔴 Note : when check packet 274 , using follow --> Tcp stream ==

#### 4. Examine the External to Internal traffic

> After determining the suspicious attack pattern its recommend to check the reverse traffic vice verse, by examine the vice verse traffic its give a better visibility over the detected activity

```
ip.src == 192.168.132.242 && ip.dst == 192.168.132.238
```

![[Pasted image 20261110171335.png]]

![[Pasted image 20261110171429.png]]
#### As per the above result we identified a stable communication with multiple PUSH requests, examining the PUSH request to identify the command execution.

