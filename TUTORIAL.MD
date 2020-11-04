# Introduction

The Buffer-Overflow is perhaps one of the most famous system attacks. The aim of this tutorial is to give some background on what makes this attack possible, then explain the attack, and finally provide some source files and binary examples for you to experiment. 

The binaries in this repo are pre-compiled for a 32 bits Linux system and have been tested with 32 bits and 64 bits Virtual Box images of [Kali](https://www.kali.org). Please read the instructions with the binaries on how to setup your system. 

**WARNING:** The tutorial attacks are only possible when you weaken the default security settings of your system. Please do not try these attacks the system you normally use and reserve them for an experimental system (e.g., Virtual Box image). 

To clone the repository within a Linux environment: 

```bash
git clone https://gitlab.com/morisset/bof.git
```

This will create a folder `bob`, which we will use below. 

# Background 

We do not provide in this page a full explanation of how system memory works. However, some important concepts to keep in mind are: 
- Programs are loaded from the hard-drive to the memory to be executed. 
- The memory is organised as a Virtual Memory, with addresses expressed in hexadecimal. All addresses we consider here are virtual. 
- The virtual memory contains a range of data: instructions coming from the program, control-flow instructions, user data, etc. Although these different types of data are normally separated, the buffer overflow attack aims at breaking the boundaries. 

If you're familiar with this, you can skip directly to the next section. Otherwise, things will hopefully be clearer once we start playing with the actual code. Don't try to understand everything below, just try to focus on the security relevant bits. In many cases, attackers are capable of compromising a system without understanding it fully: they just need to understand how to break it! 

 For now, consider the C program below, which corresponds to the file `step1.c` in the folder `bob`. This is a very simple (and boring) piece of code:

- The function `main` takes one argument from the command line (remember that in C, the first argument is always the name of the program itself, so the actual argument is the second argument), call the function `func` with that argument. 
- The function `func` takes one argument, creates a buffer of 32 bytes, copies the argument in the buffer, and print the content of the buffer. 

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>

void func(char *s){
  char buf[32];
  strcpy(buf, s);
  
  printf("you entered: %s\n", buf);  
}

int main(int argc, char *argv[]){
        if(argc != 2){
                printf("Usage: %s some_string\n", argv[0]);
                return 2;
        }

        func(argv[1]);

        return 0;
}
```



If you run this program (you should be able to run the program directly, don't try to recompile it), you'll get: 

```bash
$ ./step1 "hello world"
you entered: hello world
```

So far, so good. Let's now look at the actual program. Use the command `objdump -d step1` (the `-d` option indicates that the code should be disassembled). You should a bunch of code, including the two sections below, which correspond to the assembly code of the function `func` and `main` respectively: 

```assembly
0804843b <func>:
 804843b:       55                      push   %ebp
 804843c:       89 e5                   mov    %esp,%ebp
 804843e:       83 ec 20                sub    $0x20,%esp
 8048441:       ff 75 08                pushl  0x8(%ebp)
 8048444:       8d 45 e0                lea    -0x20(%ebp),%eax
 8048447:       50                      push   %eax
 8048448:       e8 c3 fe ff ff          call   8048310 <strcpy@plt>
 804844d:       83 c4 08                add    $0x8,%esp
 8048450:       8d 45 e0                lea    -0x20(%ebp),%eax
 8048453:       50                      push   %eax
 8048454:       68 20 85 04 08          push   $0x8048520
 8048459:       e8 a2 fe ff ff          call   8048300 <printf@plt>
 804845e:       83 c4 08                add    $0x8,%esp
 8048461:       90                      nop
 8048462:       c9                      leave  
 8048463:       c3                      ret    

08048464 <main>:
 8048464:       55                      push   %ebp
 8048465:       89 e5                   mov    %esp,%ebp
 8048467:       83 7d 08 02             cmpl   $0x2,0x8(%ebp)
 804846b:       74 1a                   je     8048487 <main+0x23>
 804846d:       8b 45 0c                mov    0xc(%ebp),%eax
 8048470:       8b 00                   mov    (%eax),%eax
 8048472:       50                      push   %eax
 8048473:       68 31 85 04 08          push   $0x8048531
 8048478:       e8 83 fe ff ff          call   8048300 <printf@plt>
 804847d:       83 c4 08                add    $0x8,%esp
 8048480:       b8 02 00 00 00          mov    $0x2,%eax
 8048485:       eb 16                   jmp    804849d <main+0x39>
 8048487:       8b 45 0c                mov    0xc(%ebp),%eax
 804848a:       83 c0 04                add    $0x4,%eax
 804848d:       8b 00                   mov    (%eax),%eax
 804848f:       50                      push   %eax
 8048490:       e8 a6 ff ff ff          call   804843b <func>
 8048495:       83 c4 04                add    $0x4,%esp
 8048498:       b8 00 00 00 00          mov    $0x0,%eax
 804849d:       c9                      leave  
 804849e:       c3                      ret    
 804849f:       90                      nop

```

No need to understand everything, a lot is just core mechanics (and it's not very intuitive!). Basically, in the left column we have the address in the virtual memory, while in the right column, we have the assembly instructions. Let us ignore most of the instructions for the moment, and focus on the following instruction in `main`: 

```assembly
8048490:       e8 a6 ff ff ff          call   804843b <func>
```

Think about the way a program executes as having a pointer that just reads instruction, one after the other, starting from the initial one (in the same you would do for a cooking recipe). In the case above, the execution pointer reaches the address `8048490`, it will execute the instruction `call 804843b` (it's actually slightly more complicated than that, but this is a good level of abstraction), which moves the pointer to the address `804843b`, which happens to be the first address of the program corresponding to `func`. The pointer will therefore execute: 

```assembly
804843b:       55                      push   %ebp
```

followed by 

```assembly
804843c:       89 e5                   mov    %esp,%ebp
```

and so on and so forth. However, when the pointer reaches the last address in `func` (`8048463`), how does it know where to return? If it just tries to execute the next address in virtual memory (`804864`), it would start the function `main` from the beginning, which is not what we want. Instead, we want to go back to the address `8048495`, which is the next address after executing the call to `func`. To keep with the cooking recipe analogy, if you are following a recipe for lasagnas and the recipe book tells you to go to another page for the white sauce recipe, you need to remember where is the lasagna recipe and at this stage you were once you're done with the white sauce. You could do that for instance by using a post-it or simply rely on your memory. 



It is the same for the computer, it relies on its memory and this is where the stack comes in. 

![Virtual Memory](Virtual Memory.png)

