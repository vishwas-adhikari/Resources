

# The Secret Life of DLLs: Memory, Modules, and Address Resolution

In our last post, we looked at the difference between an EXE (the brain) and a DLL (the helper). We established that while an EXE starts the program, it often relies on external Dynamic Link Libraries (DLLs) to do the heavy lifting.

But how do these two files actually talk to each other once the program is running? To understand cybersecurity and system internals, you have to go one level deeper into the core mechanism of the Windows operating system.

## What Happens When You Run an EXE?

When you double-click an executable, a lot happens before you see a single pixel on the screen. The Operating System (OS) doesn't just run your code; it orchestrates a complex setup:

1.  **Memory Mapping:** The OS loads the EXE file into a fresh area of RAM.
2.  **Dependency Check:** The OS looks at the EXE’s "Import Table" to see which DLLs are required (e.g., `kernel32.dll` or `user32.dll`).
3.  **Loading DLLs:** The OS finds those DLL files on the disk and maps them into the **same** memory space as the EXE.
4.  **Address Resolution:** This is the most critical step. The OS "connects" the EXE to the functions inside the DLL.
5.  **Execution:** Once the connections are made, the CPU begins running the code.

### The Concept of Address Resolution
When you write code that uses `printf()`, your EXE doesn’t actually know where that function lives. It essentially says to Windows: *"I need the 'printf' function from the library msvcrt.dll."*

Windows responds by loading `msvcrt.dll`, finding the exact memory address where `printf` starts (for example, `0x7FF...`), and handing that address to the EXE. Now, whenever the EXE calls `printf`, it knows exactly where to "jump" in memory.

## The Memory View: Everything is One

<img width="600" height="500" alt="image" src="https://github.com/user-attachments/assets/6ff0e46a-d108-4e80-8ca1-3afb5f661715" />

A key realization for any developer or researcher is that **at runtime, a DLL is not separate from the program.**

Once loaded, the DLL code sits inside the same process memory as the EXE. If you were to look at the memory of a running process, you would see sections for the EXE code, sections for the various DLLs it loaded, and shared areas like the Heap and the Stack.

Because they share the same memory space, the EXE can call DLL functions as if they were its own. This is a "jump and return" mechanism:
1.  **EXE** calls a function.
2.  **CPU** jumps to the memory address inside the DLL.
3.  **DLL** executes the code.
4.  **CPU** returns back to the EXE to continue.

## How Modern Applications Use DLLs

Almost everything an application does in Windows is a series of DLL calls. By using libraries, developers don't have to reinvent the wheel for every program.

### Common Examples:
*   **Networking:** If an app needs to connect to the internet, it calls functions inside `ws2_32.dll`.
*   **Filesystem:** To create or delete a file, it uses `kernel32.dll`.
*   **User Interface:** To show a pop-up "Message Box," it calls `user32.dll`.

### Modular Design
Large applications—like modern video games or web browsers—are built as a collection of modules. Instead of one giant, unmanageable EXE, the developers create a small EXE that acts as a coordinator for several DLLs:
*   `physics.dll` handles the movement logic.
*   `graphics.dll` handles the rendering.
*   `audio.dll` handles the sound.

This modularity allows companies to update the graphics engine by simply replacing one DLL file without having to touch the rest of the application code.

### Plugin Architectures
DLLs also allow for "Dynamic Loading." An application can choose to load a DLL *after* it has already started using a command like `LoadLibrary`. This is how "Plugins" or game "Mods" work. The main program looks in a folder, sees a new `.dll` file, and brings it into memory to extend its own functionality.

## Why This is the Foundation of Cybersecurity

Now we connect the dots. Because a DLL runs **inside** the same memory space as the EXE, it has the same permissions and privileges as that EXE.

**The Key Security Realization:**
If an EXE calls a DLL, and that DLL runs code, then whoever controls the DLL controls the execution of the process.

This is the fundamental logic behind many common attack vectors:


# DLL Injection: How Code Hijacks Trusted Processes

In the world of cybersecurity, "DLL Injection" is a term that comes up constantly. Whether you are analyzing a piece of sophisticated malware or studying how game cheats work, the core mechanism is often the same. 

But what exactly is it? 

Simply put, **DLL Injection is the act of forcing a running process to load and execute a DLL that it never intended to use.**

## Why This Works (Connecting the Dots)

To understand why this is possible, we have to look back at our mental model of process memory. We know that once a process (like `chrome.exe`) is running, it lives in its own "bubble" of RAM. Inside that bubble, it calls various DLLs to perform tasks.

Because the operating system allows a process to load libraries dynamically while it is running, an attacker can exploit this flexibility. If an attacker can successfully "inject" their own malicious DLL into a trusted process, that code will execute with the same permissions and trust level as the original program.

## Why Do Attackers Use DLL Injection?

You might wonder: *“If an attacker already has code on the system, why not just run a file called malware.exe?”* 

There are three primary reasons:

### 1. Stealth and Evasion
Security software and savvy users are very suspicious of unknown executables running in the background. However, they are much less likely to notice something unusual happening inside a trusted process like `explorer.exe` (the Windows taskbar and file manager) or a web browser. By hiding inside a legitimate process, the attacker becomes a ghost in the machine.

### 2. Privilege Piggybacking
Every process runs with a certain level of authority. If an attacker manages to inject a DLL into a process that has administrative privileges or special network access, the injected code inherits those same rights. It’s the digital equivalent of sneaking into a secure building by hiding in the delivery truck of a trusted vendor.

### 3. API Hooking and Spying
Once code is running inside a target process, it can see everything that process sees. It can intercept function calls (API Hooking), log every keystroke the user types into that specific app, or even modify the data the program is sending over the internet.

---

## The Standard DLL Injection Technique

While there are many advanced ways to inject code, the most common "Standard" technique involves a specific sequence of Windows API calls. Here is exactly how an attacker orchestrates this hijack, step-by-step.

### Step 1: Identifying the Target
The attacker first chooses a target process that is already running. This could be something common like `notepad.exe` for testing, or something critical like `lsass.exe` for stealing credentials.

### Step 2: Gaining Access (`OpenProcess`)
The attacker’s program calls the `OpenProcess` function. This requests a "handle" (a digital key) from the operating system that allows the attacker to interact with the target process's memory.

### Step 3: Clearing a Space (`VirtualAllocEx`)
You can't just throw data into another process; you need to reserve a spot for it. The attacker uses `VirtualAllocEx` to "rent" a small amount of memory inside the target process. 

### Step 4: Planting the Path (`WriteProcessMemory`)
Surprisingly, the attacker doesn't usually write the *entire* malicious code into the target memory yet. Instead, they write the **file path** of their malicious DLL (e.g., `"C:\temp\malware.dll"`) into the space they just reserved.

### Step 5: The Execution Trick (`CreateRemoteThread`)
This is the "magic" step. The attacker uses a function called `CreateRemoteThread`. This tells the target process: *"Hey, start a new thread, and for that thread, run the 'LoadLibrary' function using the file path I just wrote into your memory."*

The target process obeys. It sees a command to load a library, it sees the path to the attacker’s DLL, and it loads it—thinking it's just another standard system update or helper.

---

## The Final Result

Once `LoadLibrary` is called within the target process, the malicious DLL is mapped into memory. Almost instantly, a special function inside the DLL called `DllMain` begins to execute. 

**The memory of the target process now looks like this:**

<img width="600" height="500" alt="image" src="https://github.com/user-attachments/assets/7e8bd791-9b12-4db7-a4cc-20fe4776b311" />


The attacker's code is now a part of the "trusted" process. It has full access to the program's memory, can stay hidden from the task manager, and can perform its malicious tasks indefinitely.

## The Visual Summary

If you were to map the flow, it looks like this:

1.  **Attacker Program** finds **Target Process**.
2.  **Attacker** opens a door into the **Target**.
3.  **Attacker** reserves a "shelf" in the **Target's** room.
4.  **Attacker** puts a note on that shelf with the address of the **Malicious DLL**.
5.  **Attacker** tricks the **Target** into reading that note and loading the DLL.

## Closing Thoughts

DLL Injection is a perfect example of how the very features that make an operating system powerful and flexible can be turned against it. By understanding these low-level API interactions, we can begin to build better defenses and more effective detection tools.

In our next discussion, we’ll move from "Injection" to **"Hijacking"**—exploring how attackers can exploit the way Windows *searches* for DLLs to run their code without using a single "remote thread" API call.

***

**Vishwas S Adhikari**
*Espress0*



