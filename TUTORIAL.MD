# Introduction

The Buffer-Overflow is perhaps one of the most famous system attacks. The aim of this tutorial is to give some background on what makes this attack possible, then explain the attack, and finally provide some source files and binary examples for you to experiment. 

The binaries in this repo are pre-compiled for a 32 bits Linux system and have been tested with 32 bits and 64 bits Virtual Box images of [Kali](https://www.kali.org). Please read the instructions with the binaries on how to setup your system. 

**WARNING:** The tutorial attacks are only possible when you weaken the default security settings of your system. Please do not try these attacks the system you normally use and reserve them for an experimental system (e.g., Virtual Box image). 

# Background 

We do not provide here a full explanation of how system memory works. However, some important concepts to keep in mind are: 
- Programs are loaded from the hard-drive to the memory to be executed. 
- The memory is organised as a Virtual Memory, with addresses expressed in hexadecimal. 
- 


