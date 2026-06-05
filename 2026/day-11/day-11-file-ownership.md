# Day 11 - File Ownership Challenge

## Objective

Today I practiced Linux file ownership and group management using `chown` and `chgrp` commands on Ubuntu Linux.


# Task 1 - Understanding Ownership

Command used:


ls -l


Example output:


-rw-r--r-- 1 ubuntu ubuntu 0 May 26 devops-file.txt


Here:
- first `ubuntu` = owner
- second `ubuntu` = group

Owner controls the file.  
Group allows multiple users to access the file based on permissions.

---

# Task 2 - chown Practice

Created file:


touch devops-file.txt


Checked ownership:


ls -l devops-file.txt


Created users:


sudo useradd tokyo
sudo useradd berlin


Changed owner:


sudo chown tokyo devops-file.txt
sudo chown berlin devops-file.txt

Verified:


ls -l devops-file.txt


---

# Task 3 - chgrp Practice

Created file:


touch team-notes.txt


Created group:


sudo groupadd heist-team


Changed group:


sudo chgrp heist-team team-notes.txt


Verified:


ls -l team-notes.txt


---

# Task 4 - Change Owner and Group Together

Created file:


touch project-config.yaml


Created user:


sudo useradd professor


Changed owner and group together:


sudo chown professor:heist-team project-config.yaml


Created directory:


mkdir app-logs


Changed directory ownership:


sudo chown berlin:heist-team app-logs


---

# Task 5 - Recursive Ownership

Created directory structure:


mkdir -p heist-project/vault
mkdir -p heist-project/plans

touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf


Created group:


sudo groupadd planners


Recursive ownership change:


sudo chown -R professor:planners heist-project/


Verified:


ls -lR heist-project/


---

# Task 6 - Practice Challenge

Created groups:


sudo groupadd vault-team
sudo groupadd tech-team


Created user:


sudo useradd nairobi


Created directory and files:

mkdir bank-heist

touch bank-heist/access-codes.txt
touch bank-heist/blueprints.pdf
touch bank-heist/escape-plan.txt


Changed ownership:


sudo chown tokyo:vault-team bank-heist/access-codes.txt

sudo chown berlin:tech-team bank-heist/blueprints.pdf

sudo chown nairobi:vault-team bank-heist/escape-plan.txt

Verified:


ls -l bank-heist/


---

# Files and Directories Created

## Files
- devops-file.txt
- team-notes.txt
- project-config.yaml
- gold.txt
- strategy.conf
- access-codes.txt
- blueprints.pdf
- escape-plan.txt

## Directories
- app-logs
- heist-project
- bank-heist

---



# Commands Practiced


ls -l
chown
chgrp
chown -R


---



## Screenshots

img1.png
img2.png
img3.png
img4.png
