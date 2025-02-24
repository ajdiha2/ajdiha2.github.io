---
title: Binary Unpacking Techniques
date: 2025-02-24
modified: 2025-02-24
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
<img src="/binary-unpacking-techniques/stub.png" alt="Stub">
<figcaption>Fig 1. unpacking stub</figcaption>
</figure>

Of course, we also need to understand the Portable Executable (PE) file format, but for that I recommend you to go through the following:
* [Microsoft - PE Format](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)
* [CBJ-2005-74.pdf](/assets/pe/CBJ-2005-74.pdf)
* [Corkami - PE101](/assets/pe/pe101.pdf)
* [Corkami - PE102](/assets/pe/pe102.pdf)

Next, we’re going to see how to unpack a binary. I’m not going to cover cases where you can use generic unpacker software or the “official” unpacker of the packed binary.

# How can we identify a packed binary?

Now that we know what is a packed binary, we need to understand how can we identify one. We will list some of the well-know techniques below.

## PE Sections

First, reading the section names can gives us indication about the packed file (see the following screenshot from PE Studio, where we can see the sections named UPX0 and UPX1). Be careful as section names can be manually overwritten into something “normal” or even tricking you into thinking it’s UPX when it’s not.



##### Resources

- [Unpacking binary 101](https://sam0x90.blog/2020/06/06/unpacking-binary-101)
- [Corkami - PE101](https://github.com/corkami/pics/tree/master/binary/pe101)
- [Corkami - PE102](https://github.com/corkami/pics/tree/master/binary/pe102)
