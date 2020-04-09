# BOF-Basics
Learning material for completely newcomers in the field of BOFs 

# Buffer
A buffer in actual is a temporary memory that is allocated to a particular program to retrieve and store the data whenever it requires. 
Memory i.e. the buffer in this case is always limited and can store a finite amount of data. More specifically, the ones who are coding the program know how much data will be required by the program. That memory is known as buffer.

# Buffer Overflow 
This occurs when the buffer limit is exceeded i.e. when we put more data than what was actually expected by the buffer. This happens in programs when we try to write data into the buffer and we overrun the actual buffer size. So, the extra data overwrites the adjacent memory locations.

Suppose, we have an array in C which can be initialized as above.

``` C
char array[10];
```
Now, we have defined implicitly that this array will be storing 10 data members. Suppose it to be integers from 0-9. When we will try to add more than 10 value and will see what happens when we do it so!

Consider the following program which takes 10 values from the user on runtime and copies them into the array.

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
int main(int argc, char *argv[]) {

        char array[10];

        strcpy(array, argv[1]);

        printf("Data : %s", array);

        return 0;
}
```
On Linux, we can utilize the gcc compiler to compile this code. Once compiled we will run the executable and will supply it values 0 through 9. We can see that our program has successfully accepted the values and have printed them on the screen.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/1.png)

Now, we will supply more data to the program which it is expecting. 

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/2.png)

We can see that we have achieved "Segmentation Fault". This means that the buffer was overflowed.

GCC or GNU C Compiler has many buit-in functions by default which prevents buffer overflow vulnerabilties. While we are learning we will turn off those checks for our learnings.

# Types of Buffer Overflows
There are mainly two types of "Buffer Overflows" named as above.

1. Stack-Based Buffer Overflow
2. Heap-Based Buffer OVerflow

For the sake of basics, will only be looking onto finding and exploiting "Stack-Based Buffer Overflows".

# Security Protections Against Buffer Overflows 
The security mechanism which have been added to Windows by Microsoft are as above.

1. DEP - Data Execution Prevention
2. ASLR - Address Space Layout Randomization

## DEP - Data Execution Prevention
Programs require memory in order to run and execute. DEP is a security feature which helps protect the system memory and the memory reserved for the authorized apps to not be used unethically by any other program. DEP keeps an eye on the memory management of the programs and as soon a particular program tries to execute instructions from antoher memory location. It is stopped and the user is notified. 

In technical words, DEP enables system to set some memory locations to be non-executable. This works by setting non-execution flag on some pages like default heap, stack and memory pools. As soon as the program tries to execute instruction from these protected data pages. Is is stopped to do such activities. 

Consider that two applications are running at the same time. The OS has allocated different memory regions for both application. Suppose we have total of 20 bytes of memory. The 1st application gets the first 10 bytes and the 2nd application gets the remaining 10 bytes. So, both have different address spaces and are running in different memory locations. 

Now application 1 can not execute anything from the memory which has been allocated to the 2nd application and similarly the 2nd application can not execute anything from the memory which has been allocated to the 1st application. They solely have to complete their execution while being in their own memory limits. 

If applications do not follow the process DEP mechanism will be called and the applications will be terminated.

<b>Reference - </b> https://docs.microsoft.com/en-us/windows/win32/memory/data-execution-prevention

## ASLR - Address Space Layout Randomization
Address Space Layout Randomization is also a security feature which has been implemented by the Microsoft in order to get rid of "Buffer Overflow" vulnerabilities. ASLR as the name suggests "Space" and "Randomization" are the key words. On the memory there are different buffers (memory regions) where data is required to be stored. Those locations have their particular addresses. What ASLR does is it randomized those address of memory locations so that it becomes hard to guess which part of the program sits on which location. This happens automatically when the program is run. 

In technical words, when the program is executed it is allowed a specific process region in which it has to complete its exectution. The program needs different regios to put its data i.e. stack, heap, libraries etc. So, ASLR randomized the addresses of stack, heap and libraries positions in the allocated program's memory region. 

Let's understand it with an example.

We know that instructions are executeed one after an other. Consider the following two lines.

```C
1 - strcpy(array, argv[1]);
2 - printf("Data : %s", array);
```
We know that these are two instruction. At first, the 1st instruction will be executed and then the 2nd instruction will be executed. Suppose we wrote a complete program and ran it on Windows machine. Now the OS has allocated 60-blocks of memory (automatically) for the execution of these 2 lines. But we know that only 2 blocks are required to complete the execution of these instructions i.e. 1-60 blocks, so 1st and 2nd block will be used.

Now, what happens due to ASLR is that these two instructions are put far away from each other. Suppose the first instruction is kept in 1st memory block and the other (WE DON'T KNOW) but it has been added to randomized memory location. So if we keep our exploit code on second line. We don't know the address where it has been shifted to by the OS, so we won't be able to call it because we have to use the EIP (Instruction Pointer).

<b>Reference - </b> https://en.wikipedia.org/wiki/Address_space_layout_randomization


# Finding Buffer Overflows
There are 2 most commonly used techniques for finding "Buffer OVerflows". 

1. Fuzzing 
2. Debugging

## 1. Fuzzing
This is the most easiest way to find "Buffer Overflows". In fuzzing we try to send bigger chunks of data in order to crash the program. Once the program crashes we try to find the exact part where the program gets crashed and then we try to use it for our advantage.

There are few tools available online for fuzzing as well as some comes pre-built in Kali Linux. Meanwhile we can also write our own scripts to perform fuzzing.

___
Now we will try to find the "Buffer Overflow" vulnerability in one of our own written programs. But before that we need to recompile our code to remove the basic "Buffer Overflow" security mechanism which is implemented while we compile our program with GCC. So we will turn those protections off so that we can exploit this vulnerability. 
___

We will be using the same program but we will limit the array size to 5 to keep it as small and simple as possible.

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
int main(int argc, char *argv[]) {

        char array[5];

        strcpy(array, argv[1]);

        printf("Data : %s", array);

        return 0;
}
```
Now, we will compile this program with the default options set with GCC.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/3.png)

We can see that we were able to compile the program. Also we know that the program can only hold 5 values but in this case it is accepting more. But if we move way more further we get the segmentation fault. It is because the GCC by default set different flags which prohibit the buffers to overflow.
___
Before moving any further, we need to discuss the STACKS here. What they are and how we are getting the STACK BASED BUFFER OVERFLOWS.
___
## STACKS
Stack is a data structure based on LIFO (Last In First Out). Which means that the last value to be added on the stack will be the first to remove or release. 
```
Consider a glass of water. What happens when you pour water into it. It starts to fill up from the bottom and when to try to drink the water it gets released from the top of the glass to the bottom and at the end the glass is empty. 
```
In similar way the stacks work. Stacks only have two basic operation i.e. PUSH and POP. PUSH is used to add something on the stack which starts to be placed from the bottom and the data can be pushed until the stack is full. Meanwhile POP is used to remove the values from the STACK and it POPs the values from top to bottom. Values can be popped until there are no more values left on the stack.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/s1.png)

<b>a - </b> In figure (a) we can see that we have an empty stack. <br>
<b>b - </b> In figure (b) we pushed character A on the stack which was placed on the bottom. <br>
<b>c - </b> In Figure (c) we can see that character A was popped out of stack leaving the stack empty.

Similarly in the image we can see that one by one 3 characters were pushed onto the stack and one by one they were popped out of the stack leaving the stack empty.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/s2.png)

Now we will take a look onto how this program is actually running on the system and how we are getting the "Segmentation Fault".

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/s3.png)

___
In the above image we can see that the OS has allocated space for our program. The whole program is loaded and for the array a stack has been allocated. The Stacks on lower level works the opposite i.e. they grow from Higher Memory to Lower Memory. When we start adding the data it is being pushed onto the stack and a point comes where the Return Address stores is over written. That's the point where we get the "Segmentation Fault" because there is no instruction to be executed and the Return Address has been overwritten so the program crashes here. From here we can add our own address to make this program execute our instructions.
___
Now let's take a look on LOW LEVEL stuff. 
Whenever we run a program on the system. The data is loaded into the ADDRESS SPACE which is allocated by the Operating System. That address space have different sections and the data is loaded into those sections. 

<b>.text - </b> This section contains the actual code of the program. The code which the programmer/coder has written.<br>
<b>.data - </b> This section contains the initialized data of the program. The variables which has been assigned values.<br>
```
int a = 5;
char b = 'c';
```
<b>.bss - </b> This section contains the uninitialized data of the program. The variables whih has not been assigned the values.<br>
```
int a;
char b;
float c;
```
<b>.stack - </b> The stack is allocated where the local variables and arguments of the program are loaded.<br>
<b>.heap - </b> Heap is the extra memory allocated by the OS so that if program needs more memory it can utilize this memory. 

In the above image we can see all these sections together.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/s4.png)

We can see that it starts from Lower Memory and grows towards Higher Memory. While only the Stack goes from Higher Memory to Lower Memory. 

## Exploiting The Program
Now, we will be exploiting our previous program. We will be using GDB which is a GNU Debugger. It aid us in looking on the registers level i.e. How the program and it's values are being entertained.

1. We will turn off the ASLR protection on our system. 
```
bash -c 'echo 0 > /proc/sys/kernel/randomize_va_space'
```
2. We will compile our program with GCC while turning off the Stack Protections which are enabled by default.
```
gcc -fno-stack-protector -z execstack -no-pie sample.c -o sample
```
As here we are turning off the protections we need to know which protections are being turned off here. 
```
-z execstack : Disables Executable Stack
-fno-stack-protector : Disables Stack Canaries
-no-pie : Disables Position Independent Executables 
```
___
## 1. Stack Canaries
Canaries are the known words which are actually placed between the buffer and the control data on the stack. This is to monitor the stack based buffer overflows.
When the buffer overflows, the first data to be corrupted is the canary value. As soon as this value gets corrupted there is a failure in verification of the canary value which in turns alerts an error which is then handled by the system. 
Below figure gives an idea about how "Stack Canary" values are placed.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/6.png)

## 2. Non-Executable Stack
Sometimes the "Buffer Overflow" puts data in program's data space and on stack. These are writable memory locations. But if they are made non-executable then that data will not be executed and by so Buffer Overflow will be prevented. 

## 3. Position Independent Executables
PIE is actually the body of machine code which is placed somehwere else in the primary memory but it executes properly irrespective of being placed on different locations. PIE randomizes the instruction and function addresses everytime a particulae program is run.
___

3. We will run our program until we get the "Segmentation Fault"

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/4.png)

Here we can see that on supplying 13 values to the program we get the "Segmentation Fault". Now we will run this program with GDB in order to analyze it further. 

Program can be loaded within GDB by issuing the following command.
```
gdb -q ./sample [where ./sample is your compiled program]
```
Once loaded within GDB we can use the following command to run it with the values.
```
run AAAAA (A is 0x41 in HEX)
```
In the above image we can see that we were able to load the program in GDB and is accepting the minimum values. Meanwhile when we try to supply it 13 values we get the "Segmentation Fault".

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/5.png)

Now we can see that in GDB we are able to get the Segmentation Fault but there is one more important thing to know before we get into analyzing the binary i.e. Assembly.

The programs are run on the processor level and the process is the one which processes all these details. So we need to know a little bit about processor internal working before we move onto doing anything. 

## Registers (GPRs)
Registers are small storage areas available as a part of the CPU while General Purpose Registers are those registers which are used by the CPU during the execution. There are total of 8 general purpose registers. These reegisters have their own specific purposes but these can be utilized by the programmer as per their own choice as well. 
Meanwhile the ESP and EBP shall be avoided as much as they are used for Stack Frames and if their values are not stored things can be messed up. 

There are 8 GPRs (General Purpose Registers) in processor. 

<b>EAX - </b> Accumulator Register (Stores Return Values and for Arithmetic and Logical Operations)<br>
<b>EBX - </b> Base Register (Memory Management)<br>
<b>ECX - </b> Counter Register  (Loops and Counters)<br>
<b>EDX - </b> Data Register (Input/Output)<br>
<b>ESP - </b> Stack Pointer (Points on the top of the stack) <b>[Important]</b><br>
<b>EBP - </b> Base Pointer (Points to the base of the stack) <b>[Important]</b><br>
<b>ESI - </b> Source Index (Starting address of the source) - From where the values are required to be fetched.<br>
<b>EDI - </b> Destination Index (Address of Destination) - Where values are required to be stored.<br>

One more register which is of key interest is:
<b>EIP - </b> Instruction Pointer (Points to the next instruction which is required to be executed)<br>

The first 4 GPRs (EAX, EBX, ECX, EDX) have different sections as well. The "E" here stands for Extended in 32bits. In 64bits the "E" is replaced by "R". 64bits CPU/Processors are more common these days. These 4 GRPs can be divided as above.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/7.png)

The EAX, EBX, ECX and EDX have 2 separate portions i.e. AH, AL, BH, BL, CH, CL where "H" is for the 8bit higher addresses and "L" for 8bit lower addresses. Combining AH and AL we have AX Register (32bits) similarly BX, CX and DX. While these were extended which made them 32bits registers and nowadays they are more extended and are represented by RAX, RBX, RCX, RDX, RSP, RBP, RSI, RDI.

The most important for Stack Based Buffer Overflows are EBP and ESP. We know the basic working of Stacks. Let's learn them via these registers. 

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/8.png)

We know that stacks grows from higher memory addresses to lower memory addresses. In this case EBP is the base pointer so it has to point at the base which in this case is the highest memory location from where the stack is starting. We also know the ESP points to the top of the stack so it was set to ESP=EBP so that they both have the same starting point. As soon as the values are started to PUSHED onto the stack the ESP negates the highest value with 4bytes to move onto the next memory location. Similarly addition of 4 to the previous one is done so that it keeps on moving downwards 4bytes everytime towards the lower memory addresses as soon as the values are PUSHED onto the stack. 

Similarly when the data is being popped, 4bytes are added to the ESP like ESP+16, ESP+12, ESP+8, ESP+4 and at the end EBP=ESP which means that the stack is empty now.

___
As we now a good understanding of low level details. Now we will move onto working with the GDB.
___

## GDB (Analysing The Binary)
Once our program has been loaded into the GDB. We can see the different functions of the program by issuing the below command.
```
info functions
```
![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/9.png)

We can see different functions in the image but the key interest for us is the main function. So we can disassemble to binary code into assembly code of the main function by issuing the below command. 
```
disas main
```
![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/10.png)

Here we can see that from the top stack is being allocated due to rbp and rsp and then we can see 'strcpy' function being called and at the end we can see 'puts' function. So we have an idea that:

1. Memory is being allocated.<br>
2. Something is being copied into the buffer using 'strcpy' function.<br>
3. Values are shown on the screen by using 'puts' function.<br>

In the picture we can see the above instruction.
```
0x000000000040114c <+26>:    lea    rax,[rbp-0x5]
```
lea stands for "Load Effective Address". This is same as MOV instruction which copies the actual data from source to the destination where LEA copies the address from the source to the destination. 

At this point we will put a breakpoint. (Breakpoints are used to analyze the program step by step. Once we set the breakpoint the execution of the program will be stopped at this point so that we can analyze the state of the program at this point.

Breakpoint can be set in the following way and then we can issue data to the program.
```
b *main+26
r AAAA
```
![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/11.png)

We can see that value AAAAA is being taken and then is being pushed somwhere. We can examine particular register values. In this case we will look onto the RDX register if it has our issues data or not!
```
x/s $rdx
```
![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/12.png)

Indeed the register has our values. Now we will set another breakpoint on the above location.
```
0x40115f <main+45>        mov    rdi, rax
```
So the breakpoint can be set here in the following way.
```
b *main+45
```
After this we can use the "c" command to continue the execution of our program. 

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/13.png)

Here we can see that the lea instruction has been executed and the address which was loaded is of stack so we analyzed this stack location and found our supplied data.

From here we start exploiting the Buffer Overflow Vulnerability. We will create a pattern and will supply that particular pattern to our program. This can be done by issuing the above command. 
```
pattern create 20 (Which created this pattern - aaaaaaaabaaaaaaacaaa)
```
After that we run our program until it executes completely by issuing the above command.
```
r aaaaaaaabaaaaaaacaaa
```
On looking the data in the register we can see these values in the RBP and ESP registers.

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/14.png)

The thing which is required to be noted here is the value (aaacaaa) as this is the value which is being overwritten and making the buffer to overflow. So we can check the exact location of this data where it is placed by issuing the above command.
```
pattern search aaacaaa
```
After this we can check the RSP register value and we can see that indeed this data is present on RSP. Where the above command examines the first 20 values in string of RSP register.
```
x/20s $rsp
```

![alt text](https://github.com/d3fr0ggy/BOF-Basics/blob/master/images/15.png)

Here we can clearly see that we have got the "Buffer Overflow". Now we will move one step ahead and will see how this "Buffer Overflow" vulnerability can be utilized in exploitation.

## Exploiting The VulnServer 
There are bunch of vulnerable softwares available online by using which we can learn more about buffer overflows. In order to get started with the basics we will be focusing onto utilizing VulnServer.

At first, we need to download the VulnServer. It can be downloaded from the above mentioned URL.
```
https://github.com/stephenbradshaw/vulnserver
```

Second tool, which we require is Immunity Debugger. This tool will help us to look into the behaviour of the VulnServer when we will be looking for BOF vulnerability and will also aid us in exploiting BOF.

Immunity Debugger can be downloaded from the above mentioned URL.

```
https://www.immunityinc.com/products/debugger/
```

As far as the setup is concerned i am using Windows 10 and Kali Linux 2020 as a virtual machines in Oracle VirtualBox. You can download these from the above mentioned URLs.

- Kali Linux: https://www.kali.org/downloads/
- Windows 10: https://www.microsoft.com/en-us/evalcenter/
- Oracle VirtualBox: https://www.virtualbox.org/wiki/Downloads

