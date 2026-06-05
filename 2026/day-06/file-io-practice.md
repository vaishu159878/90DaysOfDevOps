# Day 06 – Linux Fundamentals: Read and Write Text Files

## Objective
Practice basic file handling commands in Linux.

## File Created
- notes.txt

## Commands Used

### Create File

touch notes.txt


### Write Text into File

echo "Hi" > notes.txt


### Append Text into File

echo "I am learning linux" >> notes.txt
echo "Today I am practicing read and write files in linux" >> notes.txt


### Read Full File

cat notes.txt


### Read First 2 Lines

head -n 2 notes.txt


### Read Last Line

tail -n 1 notes.txt


## Output

Hi
I am learning linux
Today I am practicing read and write files in linux


## What I Learned
- Creating files using `touch`
- Difference between `>` and `>>`
- Reading file contents using `cat`
- Viewing partial file content using `head` and `tail`
- Basic Linux file handling useful in DevOps

## Screenshot

img.png
