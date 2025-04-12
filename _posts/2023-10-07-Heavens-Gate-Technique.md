---
title : "CrackMe Analysis: Heaven's Gate and Mode Switching"
author : k4n3ki
date : 2023-10-07 00:07:00 +530
categories: [Malware Analysis]
tags: [Malware Analysis, Reverse Engineering, Windows Internals]
---

# <span style="color:red">About</span>

The term "Heaven's Gate" in the context of Windows and the <span style="color:lightgreen">WOW64</span> (Windows-on-Windows 64-bit) subsystem refers to a technique used to transition from 32-bit code running in a 64-bit process to 64-bit code. This transition involves changing the processor's mode from 32-bit (x86) to 64-bit (x64). It's important to note that this technique is specific to the Windows operating system and may not apply to other platforms.

Here’s a detailed explanation of how the Heaven’s Gate technique works and how it changes the CS (Code Segment) register:

## <span style="color:red">Background:</span>
In a 64-bit Windows environment, both 32-bit and 64-bit applications can run. However, these two types of applications have different instruction sets and operate in different processor modes.
- 32-bit applications run in the compatibility mode of the 64-bit processor and use 32-bit registers and instructions.
- 64-bit applications run natively in 64-bit mode and use 64-bit registers and instructions.


## <span style="color:red">The CS Register:</span>
In x86 architecture, the CS (Code Segment) register holds the segment selector for the code segment. It plays a crucial role in determining the current execution mode of the processor.
  
CS register values:
- x86: 0x23
- x64: 0x33

The CS register contains information about the current privilege level (Ring 0 for kernel mode and Ring 3 for user mode), as well as the base address of the code segment.

## <span style="color:red">Heaven's Gate Technique:</span>
When a 32-bit application running within a 64-bit process needs to call a 64-bit system function or interface, it must switch the processor to 64-bit mode. This is where the Heaven's Gate technique comes into play.
The technique involves executing a special syscall instruction, which triggers a context switch from 32-bit mode to 64-bit mode, effectively modifying the contents of the CS register.

## <span style="color:red">Context Switch:</span>
The Windows kernel manages the transition between modes by saving the state of the 32-bit execution environment, including the CS register, before switching to 64-bit mode. It uses the `push` and `far ret` instructions for this purpose.

When executing a far return, the processor pops the return instruction pointer from the top of the stack into the `EIP` register, and then pops the segment selector from the top of the stack into the `CS` register.

In 64-bit mode, the code within the Windows kernel or the target 64-bit function is executed. After completing the 64-bit operation, another context switch occurs to return to 32-bit mode, restoring the saved state, including the original value of the CS register.

## <span style="color:red">Usage:</span>
Heaven's Gate is typically used when a 32-bit application needs to access 64-bit system libraries or interfaces that are not available in 32-bit mode. This technique allows 32-bit applications to leverage the capabilities of the 64-bit operating system without having to run as separate processes.

In summary, Heaven's Gate is a technique used in Windows (WOW64) to transition from 32-bit code to 64-bit code by changing the processor's mode from x86 to x64. It involves a context switch managed by the Windows kernel, which saves and restores the state of the CS register and other relevant registers to ensure a smooth transition between the two modes. This enables 32-bit applications to interact with 64-bit system components when needed.

# <span style="color:red">Crackme</span>

## <span style="color:red">Info :</span>
- Author : yyk
- Language : C/C++
- Platform : Windows
- Difficulty : 3.0
- Arch : x86

Description: This contains heaven's gate. You should get KEY from 64bit area.

You can download the crackme from [here](https://crackmes.one/crackme/63b15b5333c5d43ab4ecf226).

## <span style="color:red">Walkthorough</span>

Let's upload the binary into <span style="color:lightgreen">DiE</span>, where we can observe that it contains many strings and imports from <span style="color:lightgreen">VCRUNTIME140D.dll</span>, <span style="color:lightgreen">ucrtbased.dll</span>, and <span style="color:lightgreen">KERNEL32.dll</span>.

[Capa](https://github.com/mandiant/capa) detects that it is using the Heaven's Gate technique for defense evasion.

<img alt="alt text" src="/assets/img/heaven/capa.jpg">

It simply asks for a key and prints "wrong" if the key is incorrect.

<img alt="alt text" src="/assets/img/heaven/run.jpg">

In IDA Pro, we can see that it has trouble disassembling the function in which this technique is used. Since retf is used to change the value of the CS register, it confuses IDA and prevents it from correctly defining the function.

<img alt="alt text" src="/assets/img/heaven/ida.jpg">

Let's load the binary in x32dbg and run the input to see where it is being stored. After running the binary completely and providing the input, we can go to the "Find Strings" section to see the input.

<img alt="alt text" src="/assets/img/heaven/strings.jpg">

Set a hardware access breakpoint at the address where the input is being stored. Follow the address in the dump. Right-click on the first byte of the input and select <span style="color:lightgreen">Breakpoint -> Hardware -> Access -> Byte</span>. Now, restart the debugger, pass the input, and it will break at 0x00FB17EB.

<img alt="alt text" src="/assets/img/heaven/breakpoint.jpg">

Now, we can go through the instructions to see how the technique is used.

<img alt="alt text" src="/assets/img/heaven/ret.jpg">

- First, `0x33` is pushed onto the stack.
- At 0x00FB17F3, it calls the next instruction (0x00FB17F8), and the `call` instruction will push the address 0x00FB17F8 onto the stack.
- Then, it adds 5 to the address stored at esp (0x00FB17F8 + 5 = 0x00FB17FD).
- It calls "ret far," which will pop two values from the stack: the first into EIP and the second into the CS register, causing the switch.

x32dbg crashes after the "ret far" instruction, so we will have to go through the assembly to find out the key.

> <span style="color:red">**NOTICE**</span>: One thing to note is that at 0x00FB17D1, it first pushes 00, and then at the next instruction, it pushes ecx. Later in the code, it pops the values from the stack and compares them. The reason for this is that after the switch to x64, the size of the values popped from the stack will be 8 bytes.

To get the key, we will have to disassemble the instructions according to the x64 processor. To do that, copy the bytes between the "ret far" instructions, save them to a file, and then load that file into IDA64. You will see the change in the instructions.

<img alt="alt text" src="/assets/img/heaven/cmp.jpg">

We can easily find the key, as it simply compares the characters of the key one by one. Finally, the key is "h34vEn".

In essence, the Heaven's Gate technique involves the use of a specific segment call gate to switch between 32-bit and 64-bit processor modes within the WoW64 environment. This has implications for both legitimate and potentially malicious purposes related to software compatibility and security.


## <span style="color:red">References</span>
- https://www.mandiant.com/resources/blog/wow64-subsystem-internals-and-hooking-techniques
- http://www.hexacorn.com/blog/2015/10/26/heavens-gate-and-a-chameleon-code-x8664/
- https://www.alex-ionescu.com/closing-heavens-gate/