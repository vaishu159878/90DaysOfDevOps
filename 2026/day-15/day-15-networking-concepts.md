# Day 15 - Networking Concepts: DNS, IP, Subnets & Ports

## Overview

Today I learned some important networking concepts that are used daily in DevOps and Cloud environments. I explored how DNS works, what IP addresses are, how subnetting helps organize networks, and why ports are important for communication between services.

---

# Task 1: DNS – How Names Become IPs

### What happens when we type google.com in a browser?

When we type google.com, the browser first asks a DNS server for the IP address of the website. DNS converts the domain name into an IP address. The browser then connects to that IP address and requests the webpage from the server.

### DNS Record Types

 Record  Description                              

 A       Maps a domain name to an IPv4 address    
 AAAA    Maps a domain name to an IPv6 address   
 CNAME   Creates an alias for another domain      
 MX      Specifies mail servers for a domain      
 NS      Identifies the authoritative DNS servers 

### Command Used


dig google.com


### Output


;; ANSWER SECTION:
google.com.    151    IN    A    142.250.189.142


### Observation

* A Record: 142.250.189.142
* TTL: 151

---

# Task 2: IP Addressing

### What is an IPv4 Address?

An IPv4 address is a unique address used to identify a device on a network. It consists of four numbers separated by dots.

Example:


192.168.1.10


### Public vs Private IP

 Public IP                          Private IP                     

 Accessible from the internet       Used inside private networks   
 Assigned by ISP or Cloud Provider  Assigned within local networks 

### Private IP Ranges


10.0.0.0 - 10.255.255.255

172.16.0.0 - 172.31.255.255

192.168.0.0 - 192.168.255.255


### My IP Addresses

Private IP:


172.31.46.27/20


Public IP:


3.19.223.101


Observation:

My private IP belongs to the `172.16.0.0 - 172.31.255.255` private network range.

---

# Task 3: CIDR & Subnetting

### What does /24 mean?

In `192.168.1.0/24`, the first 24 bits represent the network portion and the remaining 8 bits are used for host addresses.

### Why do we subnet?

Subnetting helps divide a large network into smaller networks. This improves organization, security, and efficient IP address usage.

### CIDR Table

 CIDR  Subnet Mask      Total IPs  Usable Hosts 

 /24   255.255.255.0    256        254          
 /16   255.255.0.0      65,536     65,534       
 /28   255.255.255.240  16         14           

---

# Task 4: Ports – The Doors to Services

### What is a Port?

A port is a communication endpoint used by applications and services. Ports allow multiple services to run on the same IP address.

### Common Ports
 Port   Service 

 22     SSH     
 80     HTTP    
 443    HTTPS   
 53     DNS     
 3306   MySQL   
 6379   Redis   
 27017  MongoDB 

### Command Used


sudo ss -tulpn


### Services Found

Port  Service      

22    SSH          
53    DNS Resolver 

---

# Task 5: Putting It Together

### What happens when we run curl http://myapp.com:8080 ?

DNS first resolves the domain name into an IP address. The request is then sent to the server's IP address using port 8080. The server processes the request and sends back a response.

### App cannot reach database at 10.0.1.50:3306

I would first check:

* Whether the database server is running
* If port 3306 is open
* Network connectivity between the application and database
* Firewall or security group rules
* Correct IP address and port configuration

---

# What I Learned

### 1. DNS converts domain names into IP addresses.

Without DNS, we would need to remember IP addresses for every website.

### 2. Private and Public IPs serve different purposes.

Private IPs are used inside networks, while Public IPs are used for internet communication.

### 3. Ports help direct traffic to the correct service.

A single server can run multiple services because each service listens on a different port.

---

# Commands Practiced


dig google.com

ip addr show

curl ifconfig.me

host google.com

sudo ss -tulpn


---

## Screenshots

img1.png
img2.png

#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham
