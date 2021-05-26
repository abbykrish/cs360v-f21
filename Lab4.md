## Handling VM exits - part 2
In the last lab we implemented the memory map portion of `handle_vmcall()`. Now we will handle the other 2 cases. 

#### Part-1 Pre-lab 
1. An easy way we can help prevent code duplication is by using macros in functions. In many of the hypercalls we have in JOS, we need to differentiate when we are running in the guest or host. Give an example of a C program that adds up all the numbers in an array when the macro is set, and multiplies them when the macro is not set. 
2. Read through the explanation of the file system [here](https://github.com/vijay03/cs360v-f20/blob/master/file_system.md). As you may read, in order to create access for the guests into the file_system, we must abstract RPC calls on top of the JOS's IPC mechanism. (Something that goes over an RPC call. Maybe runs a little python script to demonstrate the concept and bring that back to how the file system runs with host read and write. WIP) 
3. Two functions that will be important in this project are `ipc_host_send()` and `ipc_host_recv()`. They use vmcalls. Set a breakpoint at these functions and explain their workflows, to successful issue the send and recieve. 
4. After completing the lab, provide a simple diagram (you can just use function names and arrows) of the workflow of the functions that get called when a guest VM tries to read from a file. 


#### Part-2 Multi-boot map (aka e820)

Recall that JOS uses three hypercall (vmcall) instructions, the first one of which we handled in lab-3. In this lab, we will handle the other two hypercalls, which are related to host-level IPC. JOS has a user-level file system server daemon, similar to a microkernel. We place the guest's disk image as a file on the host file system server. When the guest file system daemon requests disk reads, rather than issuing ide-level commands, we will instead use vmcalls to ask the host file system daemon for regions of the disk image file. This is depicted in the image below.
![alt text](http://www.cs.utexas.edu/~vijay/cs378-f17/projects/disk-architecture.jpg)

You will need to do 4 tasks 

1. You need to modify the `bc_pgfault()` amd `flush_block()` in fs/bc.c to issue I/O requests using the `host_read()` and `host_write()` hypercalls, instead of the functions they usually use. Use the macro VMM_GUEST to select different behavior for the guest and host OS. 
2. You will also have to implement the IPC send and receive hypercalls in `handle_vmcall()` (case VMX_VMCALL_IPCSEND and VMX_VMCALL_IPCRECV), as well as the client code to issue `ipc_host_send()` and `ipc_host_recv()` vmcalls in lib/ipc.c.
3. You will need to extend the `sys_ipc_try_send()` to detect whether the environment is of type `ENV_TYPE_GUEST` or not. 
4. You need to implement the `ept_page_insert()` function.


The workflow (and hints) for the ipc_* functions is as follows:

1. `handle_vmcall()`: 
	- The `VMX_VMCALL_IPCSEND` portion should load the values from the trapframe registers. 
	- Then it ensures that the destination environment is HOST FS. If the destination environment is not HOST FS, then this function returns E_INVAL. 
	- Now, this function traverses all the environments, and sets the `to_env` to the environment ID corresponding to ENV_TYPE_FS at the host. After this is done, it converts the gpa to hva (there's a function for this!) and then calls `sys_ipc_try_send()`
	- The `VMX_VMCALL_IPCRECV` portion just calls `sys_ipc_recv()`, after incrementing the program counter.
2. `sys_ipc_try_send()` checks whether the guest is sending a message to the host or whether the host is sending a message to the guest. 
	- If the curenv type is GUEST and the destination va is below UTOP, it means that the guest is sending a message to the host and it should insert a page in the host's page table. 
	- If the dest environment is GUEST and the source va is below UTOP, it means that the host is sending a message to the guest and it should insert a page in the EPT. 
	- Finally, at the end of this function, if the dest environment is GUEST, then the rsi register of the trapframe should be set with 'value'.
3. `ept_page_insert()` uses `ept_lookup_gpa` to traverse the EPT and insert a page if not present.
	- You will find functions like `epte_present()`, `epte_addr()`, `pa2page()` to be helpful.
	- You will also need to update the reference counts of the PageInfo struct of the page you insert. There are more details about this in the comment above the function. 

Once these steps are complete, you should have a fully running JOS-on-JOS.
This marks the end of project-1.


### Submission and Deadline

Please submit your code via gitolite. To mark your submission, please have a commit labelled "Lab 4 submission. 0/1/.. slip days used.". You can modify and add a dummy file for this commit if you want. We will consider the last such commit for evaluation. The deadline for lab-4 of project-1 is:

```diff
```
