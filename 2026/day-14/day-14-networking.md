# Day 14 – Networking Fundamentals & Hands-on Checks

## Objective

Today I explored Networking Fundamentals and practiced common troubleshooting commands used by DevOps engineers to verify connectivity, DNS resolution, open ports, and HTTP responses.

---

# OSI Model vs TCP/IP Model

## OSI Model (7 Layers)

Layer            Purpose                         Examples              

7. Application   User-facing network services    HTTP, HTTPS, DNS, SSH 
6. Presentation  Data formatting, encryption     SSL/TLS               
5. Session       Session management              RPC, NetBIOS          
4. Transport     Reliable communication          TCP, UDP              
3. Network       Routing and logical addressing  IP, ICMP              
2. Data Link     MAC addressing and framing      Ethernet, ARP         
1. Physical      Transmission of bits            Cables, Hubs          

---

## TCP/IP Model (4 Layers)

Layer        Purpose                         Examples              

Application  User services and protocols     HTTP, HTTPS, DNS, SSH 
Transport    End-to-end communication        TCP, UDP              
Internet     Routing and addressing          IP, ICMP             
Link         Physical network communication  Ethernet, ARP         

---

## Protocol Mapping
 Protocol     Layer       

HTTP/HTTPS    Application 
DNS           Application 
TCP/UDP       Transport   
IP            Internet    
Ethernet/ARP  Link        

---

## Real Example


curl https://google.com


Application Layer (HTTP) → Transport Layer (TCP) → Internet Layer (IP) → Link Layer

The server response travels back through the same layers in reverse order.

---

# Hands-on Practice

## 1. Identity Check

### Command


hostname -I


### Output

172.31.46.27


### Observation

Displayed the private IP address of the EC2 instance.

---

## 2. Reachability Test

### Command


ping -c 4 google.com


### Observation

* Successfully reached google.com
* 0% packet loss
* Average latency around 8–9 ms

---

## 3. Path Check

### Command


traceroute google.com


### Observation

* Reached google.com in 6 hops
* No significant delays observed
* Demonstrates how packets travel through multiple network devices before reaching the destination

---

## 4. Open Ports Check

### Command


ss -tulpn


### Observation

Listening services found:

Port  Service      
22    SSH          
53    DNS Resolver 

SSH was actively listening on port 22.

---

## 5. DNS Resolution

### Command

nslookup google.com


### Output


Address: 142.250.189.142


### Observation

DNS successfully translated the domain name into an IP address.

---

## 6. HTTP Check

### Command


curl -I https://google.com


### Output

HTTP/2 301


### Observation

Received a valid HTTP response indicating a redirect to Google's main page.

---

## 7. Connections Snapshot

### Command


netstat -an | head


### Observation

Observed:

* LISTEN connections waiting for requests
* ESTABLISHED connections representing active communication
* TIME_WAIT connections from recently closed sessions

---

# Mini Task – Port Probe & Interpretation

## Identify Listening Port

Using:


ss -tulpn


Detected:


SSH → Port 22


---

## Test the Port

### Command


nc -zv localhost 22


### Output


Connection to localhost 22 port [tcp/ssh] succeeded!


### Result

Port 22 is reachable and SSH service is functioning correctly.

If unreachable, the next checks would be:


systemctl status ssh



sudo ufw status


ss -tulpn


---

# Reflection

## Which command gives the fastest signal when something is broken?

`ping` provides the quickest indication of basic network connectivity.

---

## What layer would you inspect if DNS fails?

* Application Layer
* DNS configuration and resolver settings

---

## What layer would you inspect if HTTP 500 appears?

* Application Layer
* Web server logs and application logs

---

## Two Follow-up Checks During a Real Incident

### Check service status


systemctl status <service-name>


### Check logs


journalctl -xe


---

# Commands Practiced


hostname -I
ip addr show
ping -c 4 google.com
traceroute google.com
ss -tulpn
nslookup google.com
curl -I https://google.com
netstat -an | head
nc -zv localhost 22
ip route


---

## Screenshots

img1.png
img2.png
img3.png
img4.png
img5.png
img6.png
