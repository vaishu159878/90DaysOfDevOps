# Day 18 – Shell Scripting: Functions & Intermediate Concepts

## Task 1: Basic Functions

### Script: functions.sh


#!/bin/bash

greet() {
    echo "Hello, $1!"
}

add() {
    echo "Sum: $(($1 + $2))"
}

greet "Vaishnavi"
add 10 20
```

### Output


Hello, Vaishnavi!
Sum: 30


---

## Task 2: Functions with Return Values

### Script: disk_check.sh


#!/bin/bash

check_disk() {
    echo "Disk Usage:"
    df -h /
}

check_memory() {
    echo "Memory Usage:"
    free -h
}

check_disk
echo
check_memory


### Output


Disk Usage:
Filesystem      Size Used Avail Use% Mounted on
/dev/sda1        50G  20G   28G  42% /

Memory Usage:
               total   used   free
Mem:            7.6G   2.1G   4.5G


---

## Task 3: Strict Mode

### Script: strict_demo.sh

#!/bin/bash
set -euo pipefail

echo "$name"

false

grep "abc" file.txt | sort


### What Happens?

#### set -u

Using an undefined variable:


echo "$name"


Output:


bash: name: unbound variable


#### set -e

When a command fails:


false


The script immediately exits.

#### set -o pipefail


grep "abc" file.txt | sort


If `grep` fails, the entire pipeline fails.

### Explanation

Flag               Purpose                                  

set -e           Exit immediately if a command fails      
set -u           Treat undefined variables as errors      
set -o pipefail  Fail pipeline if any command in it fails 

---

## Task 4: Local Variables

### Script: local_demo.sh


#!/bin/bash

local_function() {
    local name="Vaishnavi"
    echo "Inside function: $name"
}

global_function() {
    city="Pune"
}

local_function
echo "Outside function: ${name:-Not Available}"

global_function
echo "Outside function: $city"


### Output


Inside function: Vaishnavi
Outside function: Not Available
Outside function: Pune


### Observation

* `local` variables exist only inside the function.
* Regular variables are available globally.

---

## Task 5: System Info Reporter

### Script: system_info.sh


#!/bin/bash
set -euo pipefail

system_info() {
    echo "HOSTNAME & OS"
    hostname
    uname -a
}

uptime_info() {
    echo
    echo "UPTIME"
    uptime
}

disk_usage() {
    echo
    echo "TOP 5 DISK USAGE"
    du -ah / 2>/dev/null | sort -rh | head -5
}

memory_usage() {
    echo
    echo "MEMORY USAGE"
    free -h
}

cpu_processes() {
    echo
    echo "TOP 5 CPU PROCESSES"
    ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -6
}

main() {
    system_info
    uptime_info
    disk_usage
    memory_usage
    cpu_processes
}

main


### Sample Output


HOSTNAME & OS 
ubuntu
Linux ubuntu 6.x.x

UPTIME
15:30:22 up 3 days, 2 users

TOP 5 DISK USAGE
...

MEMORY USAGE 
...

TOP 5 CPU PROCESSES


---

# What I Learned

### 1. Functions Make Scripts Reusable

Functions help organize code and avoid repetition.

### 2. Strict Mode Improves Reliability

`set -euo pipefail` catches errors early and prevents unexpected behavior.

### 3. Local Variables Improve Scope Management

Using `local` prevents variables from accidentally affecting other parts of the script.

---

## Screenshots

img.png
img1.png
img2.png

