# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## Overview

Today I practiced Shell Scripting concepts such as loops, command-line arguments, package installation using scripts, and basic error handling. These concepts help automate repetitive tasks and make scripts more reliable.

---

## Task 1: For Loop

### Script: for_loop.sh


#!/bin/bash

for fruit in Apple Banana Mango Orange Grapes
do
    echo "$fruit"
done


### Output


Apple
Banana
Mango
Orange
Grapes


---

### Script: count.sh


#!/bin/bash

for i in {1..10}
do
    echo $i
done


### Output


1
2
3
4
5
6
7
8
9
10


---

## Task 2: While Loop

### Script: countdown.sh


#!/bin/bash

read -p "Enter a number: " num

while [ $num -ge 0 ]
do
    echo $num
    ((num--))
done

echo "Done!"


### Output


Enter a number: 5

5
4
3
2
1
0
Done!


---

## Task 3: Command-Line Arguments

### Script: greet.sh


#!/bin/bash

if [ $# -eq 0 ]
then
    echo "Usage: ./greet.sh <name>"
else
    echo "Hello, $1!"
fi


### Output


$ ./greet.sh Vaishnavi

Hello, Vaishnavi!


---

### Script: args_demo.sh


#!/bin/bash

echo "Script Name: $0"
echo "Total Arguments: $#"
echo "All Arguments: $@"


### Output


$ ./args_demo.sh hello world

Script Name: ./args_demo.sh
Total Arguments: 2
All Arguments: hello world


---

## Task 4: Install Packages via Script

### Script: install_packages.sh


#!/bin/bash

if [ "$EUID" -ne 0 ]
then
    echo "Please run this script as root."
    exit 1
fi

packages=("nginx" "curl" "wget")

for pkg in "${packages[@]}"
do
    if dpkg -s "$pkg" &> /dev/null
    then
        echo "$pkg is already installed"
    else
        echo "$pkg is not installed. Installing..."
        apt install -y "$pkg"
    fi
done


### Sample Output


nginx is already installed
curl is already installed
wget is not installed. Installing...


---

## Task 5: Error Handling

### Script: safe_script.sh


#!/bin/bash

set -e

mkdir /tmp/devops-test || echo "Directory already exists"

cd /tmp/devops-test || echo "Failed to enter directory"

touch test.txt || echo "Failed to create file"

echo "Script completed successfully"


### Output


Directory already exists
Script completed successfully


## Screenshots

img1.png
img2.png
img3.png
