# Day 16 – Shell Scripting Basics

## 📖 Overview

Today I began my Shell Scripting journey and learned the fundamental concepts required to automate tasks in Linux. Shell scripting helps convert manual commands into reusable scripts, making system administration and DevOps workflows more efficient.

---

## 📚 Topics Covered

### 1. Shebang (`#!/bin/bash`)

* Specifies which interpreter should execute the script.
* Usually placed at the first line of every shell script.

**Syntax:**


#!/bin/bash


---

### 2. Variables

Variables are used to store data that can be reused throughout the script.

**Syntax:**

```bash
NAME="Vaishnavi"
ROLE="DevOps Engineer"
```

### Important Notes

✅ No spaces around `=`.

❌ Wrong:

NAME = "Vaishnavi"


✅ Correct:


NAME="Vaishnavi"


---

### 3. Echo Command

Used to display text or variable values.

**Syntax:**


echo "Hello World"
echo "$NAME"


---

### 4. User Input with `read`

Allows scripts to accept input from users.

**Syntax:**


read -p "Enter your name: " NAME


---

### 5. If-Else Conditions

Used for decision making in shell scripts.

**Syntax:**

if [ condition ]; then
    commands
elif [ condition ]; then
    commands
else
    commands
fi


### Comparison Operators

Operator  Meaning                  

-gt     Greater Than            
-lt     Less Than               
-eq     Equal To                 
-ne     Not Equal To             
-ge     Greater Than or Equal To 
-le     Less Than or Equal To    

---

### 6. File Check

Check whether a file exists.

**Syntax:**


if [ -f filename ]; then
    echo "File exists"
fi


---

### 7. Service Check

Verify whether a service is active.

**Syntax:**


systemctl is-active --quiet service-name


Returns:

* `0` → Service is active
* Non-zero → Service is inactive

---

## 📝 Things to Remember

* Always use a shebang in shell scripts.
* Variables must not contain spaces around `=`.
* Double quotes allow variable expansion.
* Single quotes treat text literally.
* Spaces inside `[ ]` are mandatory.
* Use meaningful variable names.
* Test scripts with different inputs.
* Make scripts executable before running.


chmod 764 script.sh
./script.sh


---

> "Automation is not about writing big scripts. It's about solving small problems consistently."


## Screenshots

img.png
img1.png
img2.png


