### Summary

**OS Kernel**

- **Ring 0 (Kernel Mode):** This is a highly privileged mode that only certain applications can access. It allows direct interaction with hardware, managing threads, and performing other critical system functions.

- **Ring 1,2,3(User Mode):** This is the mode where most applications run. Applications in user mode can only access what the operating system permits.

When an application crashes in user mode, only that application fails. However, if the kernel crashes, the entire system crashes.
![[jagadish-malayalam.gif]]

Unlike traditional antivirus software, CrowdStrike is an EDR (Endpoint Detection and Response) solution that operates with kernel-level access. This raises the question: Can any application have kernel-level access? The answer is no. Only applications that have been thoroughly tested and signed (approved) by Microsoft are granted kernel-level privileges.

However, each time an application is updated, it must go through the testing and signing process again, which can take days. For an EDR, this delay is problematic, especially when immediate updates are needed to address a zero-day vulnerability. To address this, vendors like CrowdStrike employ a method where they get a "driver" (think of it as a gun) signed by Microsoft. Once the gun (driver) is authorized, it can load and fire bullets (configuration files) without needing additional approval. In CrowdStrike's case, they had their main driver signed by Microsoft, and all configuration files and updates were stored on the hard drive. When an update occurred, the main driver, already running in kernel mode, would load the update file, and the OS would treat it as genuine.

The issue arose when CrowdStrike pushed an update that was downloaded and stored on the device as something like "C-0000291-.sys". When the main driver, running in kernel mode, attempted to load the update, it encountered a problem due to faulty code in the .sys file, leading to a crash. The faulty code was caused by a **Null Pointer Dereference**.

**What is Null Pointer Dereference?**

A pointer is a variable that stores the memory address of another variable and is represented using the " * " symbol.

For example: `int *ptr = 001;`

So imagine a map (that is a pointer), and you have an address to go to. It is easy for you to reach that address, as the address is valid. Now, what if the address is not on the map? You don't know where to go, and you will be confused.
![[sreenivasan-udayananu-tharam.gif]]

In this case, if the application was running in user mode, it would have crashed the application only. But since the main driver was running in kernel mode, it caused a system panic, resulting in a **Blue Screen of Death (BSOD)**.

This is exactly what happened in the CrowdStrike case.

### Fun Fact About x86 Assembly

x86 assembly code is read in reverse order.

![[Pasted image 20240810003303.png]]

Here's a log from the crash. The log shows an instruction to move data from a pointer stored in `r8` to `r9`. The instruction `mov r9d, dword ptr [r8]` tells the system to locate the memory address pointed to by `r8` (with `dword` standing for double word, which is 32 bits) and move it to the `r9` register. This means the pointer `r8` is being dereferenced to get the value stored at that memory location and store it in `r9`. 

However, the problem was that `r8` was completely empty, leading to the OS crash.

### How Could This Have Been Prevented?

The code was written by someone, and while it's easy to blame the individual, human errors happen. The real issue here was the lack of proper testing before the application went into production. Normally, changes of this nature should be thoroughly tested before being deployed, which wasn’t done in this case.

Additionally, this issue couldn't be easily fixed remotely. Someone had to manually intervene to delete or rename the faulty `.sys` file.