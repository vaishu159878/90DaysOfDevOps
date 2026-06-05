# Day 08 – Cloud Server Setup: Docker, Nginx & Web Deployment

## Objective
The goal of this task was to deploy a real web server on a cloud instance using AWS EC2 and practice basic server management.

---


## Ports Configured

 Service & Port 

 SSH = 22 
 HTTP (Nginx) = 80 

---

## Commands Used

### Connect to EC2 Instance


ssh -i your-key.pem ubuntu@your-public-ip


### Update System


sudo apt update
sudo apt upgrade -y


### Install Nginx


sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx


### View Nginx Logs


sudo cat /var/log/nginx/access.log


---

## Challenges Faced

Initially, the Nginx webpage was not opening in the browser.  
I solved the issue by allowing HTTP traffic on Port 80 in the EC2 Security Group.

I also faced permission issues while connecting through SSH, which I solved using:


chmod 400 your-key.pem


---

## What I Learned

- How to launch and configure an AWS EC2 instance
- How to connect securely using SSH
- How to install and manage Nginx on Ubuntu
- Basic Docker installation and service management
- How Security Groups control server access
- How to access and save Nginx logs

---


## Screenshots

img1.png
img2.png
img3.png