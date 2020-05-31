# CS 4375 - Operating Systems

## Homework 1

### Overview

HW 1: Due Feb 3, 2020</br>
Work through questions 1-8 at the end of chapter 4 (p 35).

The code is http://pages.cs.wisc.edu/~remzi/OSTEP/Homework/HW-CPU-Intro.tgz.
Suppose we have two interactive processes with 10 instructions and 10% CPU use. We have two non-interactive processes, 10 instructions, 90% CPU. Run a simulation that
minimizes the total time required for these four processes. You will need to set some of the flags suggested in questions 1-8. ```./process-run.py -l 10:10,10:10,10:90,10:90```. Turn in the command line arguments and a screen capture of the output. Explain why you believe your result optimizes runtime

### Directory Structure

The directory structure used herein to manage documents and files is as follows:

At the top level, files describing the project meant for users to read: ```README.md```. The only other files that would be expected here is a ```.gitignore``` file, listing files and/or folders which Git should ignore and a ```.git``` file, containing git metadata. There is one subdirectory of this structure: ```/src```. **Responses to all homework questions are located in the ```/src``` subdirectory in the ```homework-1-Matthew-Montoya.md``` file.**
