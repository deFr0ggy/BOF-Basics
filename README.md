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
The security mechanism which have been added to Windows by Microsoft are as above.

1. DEP - Data Execution Prevention
2. ASLR - Address Space Layout Randomization

## DEP - Data Execution Prevention
Programs require memory in order to run and execute. DEP is a security feature which helps protect the system memory and the memory reserved for the authorized apps to not be used unethically by any other program. DEP keeps an eye on the memory management of the programs and as soon a particular program tries to execute instructions from antoher memory location. It is stopped and the user is notified. 

In technical words, DEP enables system to set some memory locations to be non-executable. This works by setting non-execution flag on some pages like default heap, stack and memory pools. As soon as the program tries to execute instruction from these protected data pages. Is is stopped to do such activities. 

This is the basic working of DEP on Windows.

<b>Reference - </b> https://docs.microsoft.com/en-us/windows/win32/memory/data-execution-prevention

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
Now, we need to compile this program.



## 2. Debugging
Debugging is the process of finding and locating errors in computer programs. There are different tools available online by using which we can perform debugging and can look for potential "Buffer OVerflows".

