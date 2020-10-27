-----------------------------------------------------------------------------
Buffer Overflow Exercises
CSC3124 - Charles Morisset
Sources:
- https://dhavalkapil.com/blogs/Buffer-Overflow-Exploit/
- http://phrack.org/issues/49/14.html

Notations:
- lines starting with the # symbol indicate commands to be executed as root. 
- lines starting with the $ symbol indicate commands to be executed as normal
user. 

NOTE: These exercises only work on the 32 bits systems (which is the Kali
version you have installed for the practicals). You can do it at home too, but
make sure you are using a 32 bits distribution.


-----------------------------------------------------------------------------

0. Initialisation

In case you haven't done it already, create a new user and add it to the sudo
group (replace <username> by a user name of your choice, e.g., bob). 

# adduser <username>
# usermod -aG sudo <username> 

The first time you want to practice with Buffer Overflow, you need to run the
script setup.sh with root privileges.
Log in as the user you just created, and untar the BO.tgz file.

In the BO folder, run the script:

$ sudo sh setup.sh

You should now be set up!

1. Step1.

Goal: create a segmentation fault.

2. Step2

Goal: Skip the evaluation of the test and execute the function allow().

3. Step3

Goal: call the function secret()

4. Step4

Goal: execute arbitrary code

5. Step 5

Goal: get a rool shell.

In order to realise this attack, you will need to execute the exploit program.
This part is not covered in the recap, and therefore can be particularly hard.
