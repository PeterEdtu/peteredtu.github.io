# Malware Analysis 101: Introduction to Windows Binary (Stage 1)
```{warning}
This Page is still under construction... Read it with caution!
```

## Description
This is the material presented in HTB France's meetup about Malware Analysis.

## Topics covered
1. Windows Executables lifecycle
2. Why Windows: Introduction to LOLBAS
3. Introduction to Windows API
4. Malware Families / Variants

The full content of the presentation will come soon!

## Payload Analysis Demo 1: Mistake or Feature?
Very basic Malware Analysis & Triage on a delivered payload made for this demo.
```{warning}
Demo Payloads are NOT available yet!
```

![demo_flarevm](img/ma_part1_demo1_flarevm.png)


### Static Analysis

**PEstudio**

![demo_pestudio](img/ma_part1_demo1_pestudio.png)


1. First look at PEstudio data: 
- Binary seems to be compiled with Microsoft Visual Linker from C++
- Size is 243200 bytes
- Entropy is 6.184 which could mean that it is not packed, nor obfuscated / encrypted

![demo_pestudio_2](img/ma_part1_demo1_pestudio_2.png)


2. Sections:
- `raw-size` and `virtual-size` are close to the same value: we can confirm this **Binary is not packed**
- Size is "small", maybe it is part of a stager or is a dropper?

![demo_pestudio_3](img/ma_part1_demo1_pestudio_3.png)

![demo_pestudio_4](img/ma_part1_demo1_pestudio_4.png)


3. Module imports:
- We can't conclude anything from the imported libraries, but we are able to gather some new information we can use to anticipate its behavior during the dynamic analysis, and since the payload may not be obfuscated and is small-sized, static data should be pretty interesting:
	- `GetCurrentThreadId`, `GetCurrentThread` and`SetThreadStackGuarantee`: can imply **Threads Manipulation**
	- `GetEnvironmentVariableW`: getting environment variables of the system can imply **Host Discovery**
	- `WriteFile`, `FindNextFileW`, `FindFirstFileW` and `NtWriteFile`: writing and searching for files can imply **Files Dropping and Files / Directories Discovery**

We can't find any imports of functions to download resources online, which can imply that this payload might NOT be a dropper or a stager. But of course, this has to be confirmed.

**Detect It Easy**

We can now inspect the binary further by looking at the strings, maybe we will find some interesting things since the Binary is not obfuscated

![demo_die_1](img/ma_part1_demo1_die.png)


`Rust` compiler! I can already tell you we are cooked for the decompiling part.

![demo_die_2](img/ma_part1_demo1_die_2)

![demo_die_3](img/ma_part1_demo1_die_3.png)


We have found an interesting long string called `search_results_.txt`, which we should expect to see in the decompiled code and during execution.

**Quick view of Ghidra Decompiler**

Rust is... something when coming to decompilation. We can always decompile the Binary to Assembly, but what is magic with **Ghidra** is that it can produce Pseudo C code to help us performing code analysis. Here... Ghidra is clearly not made for Rust.

Let's skip that for now. But here is a screenshot of me searching through the strings to find the one we have identified earlier. This Assembly experience wasn't that interesting to explore so let's skip that for now. We might need to do an entire course just for the Rust compiler. But don't worry, in part 2 we'll see more about decompiling Windows Binaries and do actual static code analysis!

![demo_ghidra](img/ma_part1_demo1_ghidra.png)


### Dynamic Analysis
Okay let's take 2 seconds to think about what we want to see / confirm:
- No external resources should be accessed (no need for wireshark or INetSim - will be covered in part 2)
- A file should be written at some point during execution
- Something with searching for files should be there
- Multi-threading or multi-processing could also be a thing
- Something with environment variables: maybe for persistence?
- And finally: what is this binary doing exacty?

**Procmon filtered on the executable name**

![demo_procmon1](img/ma_part1_demo1_procmon_1.png)


At the very beginning we can't really see something special, except the creation of new Thread, the use of environment variables and some noise.

![demo_procmon2](img/ma_part1_demo1_procmon_2.png)


Then when scrolling in the Procmon logs we find what looks like **Files / Directories Enumeration** activity. Basically, the binary is recursively searching through many folders and look for everything in them, starting at `AppData` folder. Earlier we saw a log showing us that the environment variables have been requested, it might be to get the full path of the `AppData` folder.

![demo_procmon3](img/ma_part1_demo1_procmon_3.png)


And here is the file creation event we were expecting, let's have a look at its content!

![demo_procmon4](img/ma_part1_demo1_file1.png)


This `.txt` file seems to record information from the files the binary reads during the File / Dir Enumeration and then add it to this file. The full logic behind it is a blackbox, but now we know that this binary seems to act as the first stage of a stealer, trying to gather as much secret data as possible from the system, and stores it somewhere the user would not think of. Thus, makes it ready for **Data Exfiltration**.

**Procexp**

We can only see that the process for the binary is started and then stops once it is finished. Also, we can't confirm multi-processing is a thing, also that there is any external connection.

No traces of Other processes or external connections.

**Autoruns: No persistence can be seen** 

No traces of Autoruns.

### Automated Static Analysis
Now we run Capa and see if we can confirm our hypothesis!

![demo_capa](img/ma_part1_demo1_capa.png)


The only thing we didn't catch with this basic Triage was a XOR operation (see CAPA results) which does not seem to have a big impact during execution. 

### Demo 1: Conclusion
**Verdict:** Benign (not malicious)

Why? This binary is not exfiltrating information, only doing offline recon on files and saving it in a "search file" in the Temp folder. Is it a mistake or a feature?

