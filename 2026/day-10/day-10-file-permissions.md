# Day 10 Challenge – File Permissions & File Operations

## Files Created

File Name  Method Used 

 devops.txt  touch 
 notes.txt  cat / echo script.sh  vim 

---

## Commands Used

    1 touch devops.txt
    2  cat > notes.txt
    3  vim script.sh
    4  ls -l
    5  cat notes.txt
    6  vim -R script.sh
    7  head -n 5 /etc/passwd
    8  tail -n 5 /etc/passwd
    9  ls -l devops.txt notes.txt script.sh
   10  chmod 764 script.sh
   11  ls -l script.sh 
   12  ./script.sh 
   13  chmod 444 devops.txt 
   14  ls -l
   15  chmod 640 notes.txt 
   16  ls -l notes.txt
   17  mkdir project
   18  chmod 755 project
   19  ls -l
   20  echo "test" >> devops.txt
   21  chmod 664 script.sh 
   22  ./script.sh
   23  history


---

## Permission Changes

### script.sh

Before:
```
-rw-rw-r--
```

After:
```
-rwxrw-r--
```

Permission Meaning:
- Owner → read, write, execute
- Group → read, write
- Others → read only

Numeric Permission:
```
764
```

---

### devops.txt

Before:
```
-rw-rw-r--
```

After:
```
-r--r--r--
```

Permission Meaning:
- Everyone has read-only access

Numeric Permission:
```
444
```

---

### notes.txt

After:
```
-rw-r-----
```

Permission Meaning:
- Owner → read and write
- Group → read only
- Others → no permission

Numeric Permission:
```
640
```

---

### project Directory

After:
```
drwxr-xr-x
```

Permission Meaning:
- Owner → read, write, execute
- Group → read and execute
- Others → read and execute

Numeric Permission:
```
755
```

---

## Permission Testing

### Writing to Read-Only File

Command:
```
echo "test" >> devops.txt
```

Output:
```
Permission denied
```

---

### Executing Script Without Execute Permission

Command:
```
chmod 664 script.sh
./script.sh
```

Output:
```
Permission denied
```

---


## Screenshots 

img1.png
img2.png
img3.png


---

#90DaysOfDevOps   
#DevOpsKaJosh
#TrainWithShubham