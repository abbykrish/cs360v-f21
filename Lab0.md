## Project-0

In this project, you will implement a few features to become familar with the environment and the system that you will work with for the rest of the course. 


### Introduction 
In this project, throughout the labs, you will gradually build all of the pieces required to build a hypervisor. Hypervisors provide the means of running multiple OSes managed by just one entity, a single machine. We must make sure our hypervisor does several things, such as provide OS-level isolation for users and allocating and switching out resources efficiently. 


You will use the JOS operating system running on QEMU for this project. Check the [tools page](https://github.com/vijay03/cs360v-f20/blob/master/tools.md) for an overview on JOS and useful commands of QEMU. You will work on them over the next 3 or 4 lab assignments and at the end, you will launch a JOS-in-JOS environment. You will cover many tasks, such as interfacing with the hardware with system calls, handling vmexits (similar to a context switch), and mapping the bootloader in memory. 

### Background

This README series contains some background related to project-1. Reading this document will help you understand the pieces you will be implementing on a high level as you work on project-1.

The README series is broken down into 4 parts:
1. [Bootloader and Kernel](https://github.com/vijay03/cs360v-f21/blob/master/bootloader.md) which will help you in the understanding first part of the project, namely what happens when you boot up your PC (in our case, create or boot the guest).
2. [Virtual Memory](https://github.com/vijay03/cs360v-f21/blob/master/virtual_memory.md) which will help you in understanding the second part - where we transfer the multiboot structure from the host to the guest, for the guest to understand how much memory it is allocated and how much it can use. This part also contains details about segmentation and paging, which are a good background for the project in general.
3. [Environments](https://github.com/vijay03/cs360v-f21/blob/master/environments.md) which will help you in understanding what exactly is an environment, and some details about the environment structure which is used in sys_ept_map() and the trapframe structure.
4. [File System](https://github.com/vijay03/cs360v-f21/blob/master/file_system.md) which will help you in understanding the second part of the lab, where we handle vmcalls related to reading and writing of data to a disk.

## Lab-0

For Lab-0, you will first set up your working environment and then implement code to demonstrate understanding of the codebase. 


## 1. Getting Started

You will need to use your laptop for this project. Because the lab computers do not have easy root access, and we are trying to write a hypervisor, it is easiest to use a class VM for a standardized project experience. 


For lab-0, you will use a virtual machine with the Debian Jessie operating system. Follow the instructions below for
1. Setting up a VM and other essentials
2. Running JOS code for project-1

#### Setting up a Virtual Machine and Other Essentials

1. Download the compressed [VM image](https://www.cs.utexas.edu/~vijay/teaching/project1.tar.gz) (3.4 GB) on your personal laptops (with QEMU and KVM enabled). 

2. Download a Virtual Machine software: VMWare Fusion [here for students](https://my.vmware.com/web/vmware/evalcenter?p=fusion-player-personal) or Virtual Box. 
3. Load ova into whichever software you are using [link](https://help.okta.com/en/prod/Content/Topics/Access-Gateway/deploy-vmwareworkstation.htm)
4.  Begin running VM. To log into the VM, you can either use
username: user
password: user

username: root
password: root

5.  Run `ip addr` to retrieve the IP address of the machine. 
6.  Open a new terminal tab on your computer, and ssh into the machine. You can do this with `ssh -p 22 user@IP_ADDRESS`
7.  You can locate into the project folder now. It should be available on your VM, with the necessary libraries. 

#### Running JOS VMM

Compile the code using the command:
```
$ make clean
$ make
```
Please note that the compilation works with gcc version <= 5.0.0. The Makefile uses gcc 4.8.0, which is present in the gilligan lab machines. Please install gcc-4.8 if you don't have it installed already.

You can run the vmm from the shell by typing:
```
$ make run-vmm-nox
```

## 2. Coding Assignment 

### Tracing code with GDB and pointer arithmetic review 

To begin, we'd like you to get familar with using GDB. GDB is a great tool for debugging system level programs because it gives you the ability to see what is happening at a low level. 

First, set up GDB for the JOS computers using the guide founds in the [tools page](https://github.com/vijay03/cs360v-f20/blob/master/tools.md). Additionally, here is a reference sheet for [GDB commands](https://users.ece.utexas.edu/~adnan/gdb-refcard.pdf)

In order to run JOS in GDB, run 

```
gdb .\ yadda yadda 
r - args 
```

First, place a breakpoint at `env_pop_tf(struct Trapframe *tf)` in `kern/env.c`
This function restores the register values in the Trapframe with the 'iret' instruction.
An IRET instruction returns from the OS to the application program which made the system call. The IRET also makes a change back to User Mode.

Observe the register values, and note them down. 

1. Which register contains the return value? What is the value you see? Is this an address or a constant value? 

2. Which register contains the next assembly instruction to execute? What is the value you see? Is this an address or a constant value? 

Using the answer from the previous question, what are the next 5 instructions that will be executed? What are the hex codes of those functions? 

As you may notice, the function calls a c function `__asm __volatile()`. The asm statement allows you to include assembly instructions directly within C code. The `volatile` keyword simply tells the assembler not to optimize this instruction away. 

Using the register values you observed, list what the changed registers will look like after the instructions execute. Note what the difference is between the different move instructions.

```
movq %0, %rsp
movw (%rsp), %es
addq $16, %rsp
addq $16, %rsp
```

You can check your work using the `stepi` functionality in GDB, which will execute one instruction at a time. 

Now, if I were to call this instruction:
```
movq $0, (%rsp)
```
Why might this throw an error? 


### Track number of runs for an env

NOTE: All Lab 0 hints are in the codebase `Hint, Lab 0`. It is important to visit each one, for they all indicate a place where you must make an edit. 

In JOS the terms "environment" and "process" are interchangeable - they roughly have the same meaning. We introduce the term "environment" instead of the traditional term "process" in order to stress the point that JOS environments do not provide the same semantics as UNIX processes, even though they are roughly comparable.

[This guide](https://github.com/vijay03/cs360v-f21/blob/master/environments.md), linked at the beginning of the project, provides an in depth introduction into environments. 

In this part of the project, we want to be able to keep track of number of times environment has run, as an important piece of metadata. Add a new field to the struct declaration of `Env` called `env_runs` in order to track this information. 

Again, check all of the `Hint, Lab 0` places to update and use the `env_runs` variable!!

### Fix implementation of envid2env() in inc/env.c

Throughout these Labs in project_1, you will have to use a helper function, envid2env, to retrieve the Env struct, that contains metadata about an environment. 

Right now, this function has a few bugs in it. 
1. If envid is zero, the function should return the current environment.
2. It incorrectly looks up the env in the envs array. 
3. It incorrectly fills the **env_store with a null value. 

Using env macros and information in inc/env.h and your knowledge of C references, fix these three bugs. 

## Hints

In all lab assignments in project-1, the functions you will be implementing might have hints on how to implement them as comments above. So please pay attention to the comments in the code.

## Submission Details

Submissions will be handled through gitolite.
Commits made until midnight on the day of the submission deadline will be used for grading.

## Contact Details

Reach out to the TAs in case of any difficulties.
