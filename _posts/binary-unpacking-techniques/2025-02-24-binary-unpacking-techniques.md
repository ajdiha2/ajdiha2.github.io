---
title: Binary Unpacking Techniques
date: 2025-02-24 03:30:12 +0330
modified: 2025-02-24 12:09:50 +0330
tags: [upx, unpack]
---

[Original article](https://sam0x90.blog/2020/06/06/unpacking-binary-101) changed by me:

This is a quick blog post about how to unpack your first binary, hope you’ll learn something 🙂 I tried to make this article not too long so the techniques covered are fairly basics, but this should get you on track to discover more advanced unpacking techniques.

# What do we mean by “packed” binary?

First of all, we need to understand what a “packed” binary is. Basically, a packed binary is an executable file that has been “packed” by a “packer”. The end… 

Using a packer allows the author to concatenate one application (code and data) into a compressed file, which contains a routine to unpack the original executable and run it in memory. This has many purposes:

* Initially used for compression, i.e. to reduce the total size of the executable
* To prevent people from reverse engineering the file. This is done to protect things like license or protect intellectual property
* Finally, this is used by malware author to lower down the detection rate of security tools (AV, EDR, etc.) and also to make the analysis harder (or at least waste some of our time) by incorporating things like anti-reverse techniques and/or obfuscation.

Therefore, packing has both malicious and legitimate use cases, that’s why you’ve probably heard of commercial or open source packers such as the famous UPX. That’s the one we will see in a moment.

Here is a visual representation of the high-level process of unpacking to better understand the concept. Note that the packed file is also called “stub”.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/stub.png" alt="Stub">
<figcaption>unpacking stub</figcaption>
</figure>

Of course, we also need to understand the Portable Executable (PE) file format, but for that I recommend you to go through the following:
* [Microsoft - PE Format](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)
* [CBJ-2005-74.pdf](/assets/posts/pe-binary-format/CBJ-2005-74.pdf)
* [Corkami - PE101](/assets/posts/pe-binary-format/pe101.pdf)
* [Corkami - PE102](/assets/posts/pe-binary-format/pe102.pdf)

Next, we’re going to see how to unpack a binary. I’m not going to cover cases where you can use generic unpacker software or the “official” unpacker of the packed binary.

# How can we identify a packed binary?

Now that we know what is a packed binary, we need to understand how can we identify one. We will list some of the well-know techniques below.

## PE Sections

First, reading the section names can gives us indication about the packed file (see the following screenshot from `PE Studio`, where we can see the sections named UPX0 and UPX1). Be careful as section names can be manually overwritten into something “normal” or even tricking you into thinking it’s UPX when it’s not.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/pestudio-sections.png" alt="pestudio">
<figcaption>pestudio sections</figcaption>
</figure>

We can also use tools such as CFF Explorer or PEiD that will try to automatically determine what kind of packed file we are facing. Below we can see that CFF Explorer found that the file has been packed with UPX v3.0.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/cffexplorer-sections.png" alt="CFF Explorer">
<figcaption>CFF Explorer sections</figcaption>
</figure>

## Import Table

Another good technique to determine if the file is packed, is to have a look at the import table, which should be relatively small as it only uses functions to “decrypt” or unpack the original file. Here after, we can see the import table of a UPX packed file shown in `PE Studio`, which is quite short.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/pestudio-import-table.png" alt="pestudio">
<figcaption>pestudio import table</figcaption>
</figure>

You might also get a warning in some debuggers, if the file is packed or see only small amount of code being recognized by the disassembler.

Of course there are more techniques that you can use to determine if the file is packed such as the rights of each section (having READ, WRITE and EXECUTE shouldn’t be the case), having a entry point in another section than the first one, or checking the entropy of the sections to spot encryption, etc.

# How can we find the original entry point?

Ok, we’ve seen how we can identify (simple) packed binaries, now it’s time to unpack it, but first we need to retrieve the original entry point. From now on, the example used is the “unpackme_UPX” binary that you can found by googling around.

So the goal here will be to partially execute the binary, indeed, we will need the packed binary to go through its unpacking/decrypting routine and stop the execution at the entry point of the original binary. This what we call : Finding the Original Entry Point (aka OEP).

Again there are tons of techniques available to determine the original entry point, but here are some of them.

## Browsing

“Browsing” the code manually to find the OEP of well-known compilers. For example, the first API call of the Microsoft Visual C++ version 6 compiler is the “GetVersion” API call, therefore we can try to find it manually by browsing the code. Here below is a example of how the pattern looks like in OllyDbg for this particular compiler.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/ollydbg-getversion.png" alt="OllyDbg">
<figcaption>OllyDbg KERNEL32.GetVersion</figcaption>
</figure>

Without knowing patterns from compilers, visually browsing the code can still be effective to retrieve the original entry point. For example, when we open the “unpackme_UPX” in a debugger such as x32/x64dbg or OllyDbg binary and scroll down a bit we can observe a “final” jump like in the screenshot below. This is called a “tail jump” which is referring to the moment where the stub is transferring the execution to the OEP.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/ollydbg-tail-jump.png" alt="OllyDbg">
<figcaption>OllyDbg tail jump</figcaption>
</figure>

If we follow that jump (click on the line and hit “Enter”) we will arrive in the first section of the binary, which is still packed because we moved there statically. Therefore the disassembler of your debugger will not be able to retrieve the original code (see screenshot below).

<figure>
<img src="/assets/posts/binary-unpacking-techniques/ollydbg-empty-jump.png" alt="OllyDbg">
<figcaption>OllyDbg jump to empty code</figcaption>
</figure>

In order to recognize the code, we will need the binary to execute its unpacking routine and stop the execution at the OEP. What we can do is placing a software breakpoint (F2 in x32dbg) on this last jump (address 0046DEFC in our example above) and single step into (hit “F7” button) after the execution stopped at this breakpoint.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/ollydbg-tailjump-bp.png" alt="OllyDbg">
<figcaption>OllyDbg tail jump breakpoint</figcaption>
</figure>

Once the software breakpoint is placed we can hit run (or “F9” button) to start the execution. EIP (the instruction pointer) will stop on the tail jump, now we can single step into (hit “F7” button) to follow the jump.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/ollydbg-unpacked-oep.png" alt="OllyDbg">
<figcaption>OllyDbg unpacked OEP (Original EnterPoint) </figcaption>
</figure>

There you go, we arrive at the OEP and we can observe the particular pattern from Microsoft Visual C++ version 6 compiler and its first API call “GetVersion” that we saw in the OllyDbg screenshot earlier.

## ESP trick

Another quick technique to find the OEP is what we call the “ESP trick” or spotting a restoration of registers from the stack using PUSHAD and POPAD assembly instructions. Basically here, the PUSHAD instructions is performed at the beginning of the execution to push all general-purpose registers to the stack, then the unpacking routine starts and finally the POPAD instruction is called to retrieve the registers before the execution is passed to the original binary/code.
We could leverage this to stop the execution at the POPAD instruction. Let’s check our example “Unpackme_UPX” again.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/pushad.png" alt="Debugger">
<figcaption>PUSHAD: push all general-purpose registers to the stack</figcaption>
</figure>

Here, we are at the entry point of our packed sample and we can see that the first instruction is PUSHAD. We can single step this instruction and observe the stack.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/pushad-stepinto.png" alt="Debugger">
<figcaption>PUSHAD step into</figcaption>
</figure>

If we look at the stack (window on the bottom right) we can see that the value stored in the general-purpose registers (window on the top right) have been pushed to the stack. Now let’s place a breakpoint on memory access to stop the execution when this exact “place” in memory will be reached, meaning that the POPAD instruction has been executed to retrieve the content from the stack back to the registers.
In order to do that, we can right click on the address of the stack to “follow in dump” like below.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/stack-follow-in-dump.png" alt="Debugger">
<figcaption>Stack follow in dump</figcaption>
</figure>

Now we can see in the dump (window on the bottom left), which in our case represents the stack, the values from the registers (see screenshot below).

<figure>
<img src="/assets/posts/binary-unpacking-techniques/pushad-result.png" alt="Debugger">
<figcaption>PUSHAD result</figcaption>
</figure>

Let’s now right click on this address and setup a hardware breakpoint on memory access (software breakpoint cannot be used for memory access).

<figure>
<img src="/assets/posts/binary-unpacking-techniques/memory-hardware-access-bp.png" alt="Debugger">
<figcaption>Memory hardware access breakpoint</figcaption>
</figure>

Here we select “Hardware, Access” and then “Dword” to specify the 4 first bytes, which represent the size of one register.

Then, we execute the binary by hitting “F9” button in x32dbg and observe the execution being stopped at the hardware breakpoint.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/memory-hardware-bp-hit.png" alt="Debugger">
<figcaption>Memory hardware breakpoint hit</figcaption>
</figure>

Now that we stopped we can see that we arrive almost at the same place as browsing the code before. So in this specific case it was easier to just simply browse the code manually, but this trick can be very useful on more complex packed binaries.

## Further exploration

Other techniques exist of course, such as leveraging DEP for access violation when executing code in the first section (which we assume contains the OEP) or tracing back call stacks, etc.


# How can we unpack the binary?

Now that we found the OEP, we can unpack the binary to ease our analysis.

With the instruction pointer EIP on the OEP, we can now dump the process from memory to disk file. There are many different ways of doing it, I will show you how to do it using x32dbg plugin “Scylla”, but you can do it with the plugin Ollydump from OllyDbg or other tools.

## Scylla plugin

After opening Scylla plugin, and if your EIP is already at the OEP, it will be populated correctly in Scylla, otherwise you can just change it to the OEP address manually, which is, in our case, 004271B0. Then we can hit “IAT Autosearch” to get back the import address table for the unpacked executable file. Then, we need to hit the “Get Imports” button to retrieve all imports of the unpacked file. Finally, we hit “Dump” button to dump the process from memory to a file on disk.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/scylla-iat.png" alt="Debugger">
<figcaption>Scylla : IAT Autosearch / Get Imports / Dump</figcaption>
</figure>

We might think we’re done, but there is one last final step. We need to fix the import table as the binary we’ve just dumped will not work. In order to do that, we can click on “Fix Dump” button and select the binary we’ve just dumped.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/scylla-fix-dump.png" alt="Debugger">
<figcaption>Scylla : Fix Dump</figcaption>
</figure>

You will see a log message saying “Import Rebuild success”. Now you can run the binary and it should work. You can also re-open that binary in x32dbg and check if we arrive at the OEP directly.

You can also use tools such as Import REConstructor as sometimes plugins can fail to fix the import table.

# Final analysis
After unpacking, we can re-analyze our binary in `PE Studio`. We can immediately see that the import table is much bigger than initially, thus giving us some quick hints on what the binary will be doing. The strings tab has also more results populated, etc.

<figure>
<img src="/assets/posts/binary-unpacking-techniques/pestudio-complete-import-table.png" alt="Debugger">
<figcaption>pestudio correct import table</figcaption>
</figure>

# Some “free” tips to conclude

* When unpacking or analyzing malware you might face anti-reverse techniques that we didn’t cover in this post. I might create one specifically on it, but in the meantime you can have a look at this: [The “Ultimate”Anti-Debugging Reference](/assets/posts/binary-unpacking-techniques/The_Ultimate_Anti-Reversing_Reference.pdf)
* If you want to setup a malware analysis environment, I recommend to check out the Flare-VM from FireEye. It will install all the necessary tools needed for malware analysis: [flare-vm](https://github.com/fireeye/flare-vm)
* Also the REMnux distribution from Lenny Zeltser: [REMnux](https://remnux.org/)
* Usually, it’s good to have a Windows 7 machine (or even Windows XP for old stuff), but you would be mostly fine with a Windows 10 machine as well.
* Installing VM guest tools or not? Always an interesting discussion to have with malware analyst. I’ll give you the best answer I had so far: If you’re dealing with common malwares you should be fine by installing them, but if it’s something completely new or if there’s a risk of having an exploit evading hypervisor then setup a VM without guest tools installed would be preferable.
* Emotet is a malware packed several times so it could be a good challenge, I have to check it out myself. You can find samples here and search for the “emotet” tag: [app.any.run](https://app.any.run/submissions) and also here: [bazaar.abuse.ch](https://bazaar.abuse.ch/browse/tag/Emotet/)

Hope you enjoyed, I’ll probably try to write more posts on my RE journey, feel free to reach out to me on twitter [Sam0x90](https://twitter.com/Sam0x90)

Hacky reverse!


##### Resources

- [Unpacking binary 101](https://sam0x90.blog/2020/06/06/unpacking-binary-101)
- [Corkami - PE101](https://github.com/corkami/pics/tree/master/binary/pe101)
- [Corkami - PE102](https://github.com/corkami/pics/tree/master/binary/pe102)
