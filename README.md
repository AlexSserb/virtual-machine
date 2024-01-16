# Virtual machine

## Table of contents
* [Architecture](#architecture)
* [Tools and technologies](#tools-and-technologies)
* [Byte code structure](#byte-code-structure)
* [Byte code example](#byte-code-example)
* [Program execution](#program-execution)

<a name="architecture"></a>
## Architecture

A Virtual Machine (VM) is a program that is designed to interpret the bytecode of other programs based on a language.
This virtual machine interprets the code that is passed to it in a file line by line.

The following RISC-like architecture was used to develop the virtual machine:
* PSW = IP + Flags = 16 + 16 = 32 bits
* Memory: words – 32 bits, address size – 16 bits
* Data types:
  - Signed integers – 1 word
  - Unsigned integers - 1 word
  - Fractional – 1 word
* Address registers – 256 pieces of 16 bits each
* Command structure: 1 word = 32 bits. Consists of:
  - operation code – 8 bits
  - regiter 1 – 8 bits
  - regiter 2 – 8 bits
  - regiter 3 – 8 bits
* Command for loading the address register: 1 word = 32 bits. Consists of:
  - operation code – 8 bits
  - register 1 – 8 bits
  - address (constant) – 16 bits
* Unconditional transitions:
  - r1 is the type of transition. There are four types:
     - 0: direct, IP = address (constant) in the command
     - 1: direct indirect, IP = the value in memory that the address (constant) in the command points to
     - 2: direct indirect register, IP = reg2 + reg3
     - 3: relative, IP = IP + offset; offset = constant in the command
* Conditional transitions have the same structure, but also check flags
* Subroutine call – the return address is stored in reg1
* Return from subroutine is an unconditional direct transfer (address in reg1)
* A stack is used for recursive subroutine calls. The stack is simulated by the last 16 registers (240 to 255)

<a name="tools"></a>
## Tools and technologies
* C++
* Linux Debian (with Oracle VM Virtual Box)
* Git
* Codeblocks IDE


<a name="Byte code structure"></a>
## Byte code structure

The bytecode file consists of the following parts:
1. The mark "a" and the address of the beginning of writing the bytecode to the VM memory
2. The main subroutine. Consists of:
  - Definitions of variables
  - Instructions
3. The "e" label and the address of the start of the program
4. Subroutines. Each subroutine consists of:
  - Definitions of variables
  - Instructions
  - In the end of subroutine shoud be command "ENDP" with code 54

According to the assignment, it was necessary to develop a bytecode accessible for human development and editing. Therefore, for convenience, the following has been added:
* The virtual machine processes text files in which the bytecode is written line by line
* Three marks are used to define variables: 
  - "i" for integers
  - "u" for unsigned integers
  - "f" for fractional numbers
* You can place comments in the file using the "#" symbol. The virtual machine will not execute them in any way

Thus, you can write the bytecode yourself and add comments immediately after the commands:
```
a 0 # Comment as the title of the program
# Comment 1
k command_code reg1 reg2 reg3  # Comment 2
k command_code reg1  # Comment 3
e 0
```

The same code without comments:
```
a 0
k command_code reg1 reg2 reg3
k command_code reg1
e 0
```

<a name="byte_code_example"></a>
## Byte code example
An example of a bytecode for sequentially reading three numbers and outputting their factorials:
```
a 0 # A program for calculating the factorials of three numbers
u 1 # UINT start 1
u 1 # UINT res 1
u 1 # UINT const1 1
# Three procedure calls
k 51 14 # Call address
k 51 14
k 51 14
e 8 # End of the main subroutine, which starts from cell 8
# The procedure for calculating the factorial
k 23 1 0 # LOAD R1 start
k 23 2 6 # LOAD R2 user_value
k 23 3 2 # LOAD R3 res
k 23 4 4 # LOAD R4 const1
k 43 2 # READU R2
k 27 1 2 # CMPU R1 R2
k 6 3 8 # JGU 3 8
k 33 3 3 1 # MUL R3 R3 R1
k 40 1 # INC R1
k 1 3 -8 # JMP 3 -8
k 21 3 # PRINTU R3
k 50 1 4 # LOADRV R1 R4
k 50 3 4 # LOADRV R3 R4
k 54 # ENDP

```
The virtual machine, executing the data bytecode, will prompt us to enter three numbers. After entering the number 5, 120 will be output (5! = 120). After entering 3, 6 will be output (3! = 6). After entering 10, 3628800 will be output (10! = 3628800).

The result of the program execution:
```
5           // input 1
120         // result 1
3           // input 2
6           // result 2
10          // input 3
3628800     // result 3
```

<a name="program_execution"></a>
## Program execution

To run the program, you need to create an executable file and execute it through the terminal. 

Since the virtual machine interprets the bytecode file, the path to the bytecode file must be added to the command to execute.

The command to run consists of the path to the VM executable file and the path to the bytecode: 
```bash
$ /home/user/path_to_executable_file/VirtualMachine9 /home/user/path_to_bytecode_file/file.txt
```
