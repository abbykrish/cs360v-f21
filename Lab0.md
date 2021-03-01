## Project-0

In this project, you will implement a few features to become familar with the environment and the system that you will work with for the rest of the course. 

You will use the JOS operating system running on QEMU for this project. Check the [tools page](https://github.com/vijay03/cs360v-f20/blob/master/tools.md) for an overview on JOS and useful commands of QEMU. You will work on them over the next 3 or 4 lab assignments and at the end, you will launch a JOS-in-JOS environment. 

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

You may use your laptops / computers for this project. Please enable qemu-kvm on your machines. Alternatively you can also use any of the following (gilligan) CS machines. As you need access to KVM module for this project, you cannot use other CS machines.
- ginger
- lovey
- mary-ann
- skipper
- the-professor
- thurston-howell-iii

For lab-0, you will use a virtual machine with Ubuntu 16.04 operating system. Follow the instructions below for
1. Setting up a VM and other essentials
2. Running JOS code for project-1

#### Setting up a Virtual Machine and Other Essentials

1. Download the compressed [VM image](https://www.cs.utexas.edu/~vijay/teaching/project1.tar.gz) (3.4 GB) on CS gilligan machines or your personal laptops (with QEMU and KVM enabled). The uncompressed VM image (8.8 GB) is available for download [here](http://www.cs.utexas.edu/~soujanya/project1-vm.qcow2).
```
$ wget https://www.cs.utexas.edu/~vijay/teaching/project1.tar.gz
```


2. Now start up a VM that listens on a specific port using the following command. To avoid contention over ports, use `<port-id> = 5900 + <team-number>`. For example, if your group-id is 15, your port-id will be 5915.
```
$ qemu-system-x86_64 -cpu host -drive file=<path-to-qcow2-image>,format=qcow2 -m 512 -net user,hostfwd=tcp::<port-id>-:22 -net nic -nographic -enable-kvm
```

3. On another terminal, connect to the VM using the following command. On connecting, enter the password as `abc123`.
```
$ ssh -p <port-id> cs378@localhost
```

If you see an error that says something similar to `Could not set up host forwarding rule 'tcp::5905-:22'`, there may be some other program that is listening to the same port as yours. 

To figure out if this is the case, 
`lsof -i:5905` or `nc -l -p  5905` will tell you if something is running on that port.


4. Copy your public *and* private ssh keys from the CS lab machine or from your local machine into the VM.
Alternatively, you can generate a new key-pair on the VM using `ssh-keygen -t rsa`. You should send the public key in the VM to the TAs.
```
$ scp -P <port-id> $HOME/.ssh/id_rsa.pub cs378@localhost:~/.ssh/id_rsa.pub
$ scp -P <port-id> $HOME/.ssh/id_rsa cs378@localhost:~/.ssh/id_rsa
```

5. Verify that you have gdb 7.7 and gcc 4.8 in the VM. Also cross check that you have python-3.4 installed or in your $HOME directory. In case you need to install any of them, follow the instructions on the [installations](https://github.com/vijay03/cs360v-f20/blob/master/installation.md) page. Note that to exit from QEMU VM press `Ctrl a` then `x`.

6. We will be using gitolite to manage access for different groups. Inside the VM, clone the gitolite repo using
```
git clone cs378-vijay@git.cs.utexas.edu:group<groupnumber>-project1
```
The repo will contain a project-1 folder which has the source code for the project.
You can use git to commit and push your code changes to the gitolite repo.
You can also get started with the lab without having access to gitolite. Find the project-1 code [here](https://github.com/vijay03/cs360v-f20/blob/master/project-1.tar.gz) and untar it and get access to the project-1 code.

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

In this part of the project, we want to be able to keep track of number of times environment has run, as an important piece of metadata.

Add a new field to the struct declaration of `Env` called `env_runs` in order to track this information. 

### Fix implementation of envid2env() in inc/env.c

Throughout these Labs in project_1, you will have to use a helper function, envid2env, to retrieve the Env struct, that contains metadata about an environment. 

Right now, this function has a few bugs in it. 
1. If envid is zero, the function should return the current environment.
2. It incorrectly looks up the env in the envs array. 
3. It incorrectly fills the **env_store with the correct env reference. 

Using env macros and information in inc/env.h and your knowledge of C references, fix these three bugs. 

## Hints

In all lab assignments in project-1, the functions you will be implementing might have hints on how to implement them as comments above. So please pay attention to the comments in the code.

## Submission Details

Submissions will be handled through gitolite.
Commits made until midnight on the day of the submission deadline will be used for grading.

## Contact Details

Reach out to the TAs in case of any difficulties.
