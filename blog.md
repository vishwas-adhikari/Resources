

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


This is the fourth post in our series on Windows internals and security. In our last discussion, we explored **DLL Injection**, which is an active, "noisy" way of forcing code into a process. 

Today, we look at its quieter, more deceptive cousin: **DLL Hijacking.**

***


# DLL Hijacking: The Art of Being Found

While DLL Injection involves actively forcing a process to load malicious code using specific Windows APIs, **DLL Hijacking** is a passive technique. It doesn't force anything. Instead, it relies on the natural way Windows searches for files. 

If DLL Injection is like breaking into a building and planting a bug, DLL Hijacking is like replacing the building's legitimate directory with a fake one so that the employees unknowingly walk into the wrong room.

## How Windows Loads a DLL

When a developer writes an application, they often need to load a library. Sometimes, they use the full path (e.g., `C:\Windows\System32\user32.dll`). But frequently, for the sake of simplicity or portability, they only specify the filename:

`LoadLibrary("helper.dll");`

When Windows sees this, it doesn't immediately know where the file is. It has to go looking for it. To do this, it follows a pre-defined **Search Order**:

1.  **The Application’s Directory:** The folder where the `.exe` is located.
2.  **System Directories:** Locations like `C:\Windows\System32`.
3.  **The Windows Directory:** `C:\Windows`.
4.  **The Current Working Directory.**
5.  **Directories in the System PATH environment variable.**

The most important rule is this: **Windows loads the first matching DLL it finds.** It doesn't check if the DLL is "official" or "safe"; it simply assumes that if the name matches, it's the right file.

## The Opportunity for Hijacking

The vulnerability exists because the **Application’s Directory** is checked first. 

If a program expects to find a system DLL in the `System32` folder, but an attacker places a malicious DLL with the *exact same name* in the program’s local folder, Windows will find the attacker's version first and stop looking. 

<img width="600" height="500" alt="image" src="https://github.com/user-attachments/assets/9843de62-3192-4a6c-9227-fc019c14c52d" />


### A Step-by-Step Scenario:
Imagine an application located at:  
`C:\ProgramFiles\VideoEditor\editor.exe`

When you launch `editor.exe`, it tries to load a library called `helper.dll`. Normally, this file lives in a deep system folder. 

However, an attacker puts their own malicious version of `helper.dll` right next to the editor:  
`C:\ProgramFiles\VideoEditor\helper.dll` **(Malicious)**

**The Result:**
1.  The user runs `editor.exe`.
2.  Windows looks in `C:\ProgramFiles\VideoEditor\` for the DLL.
3.  It finds the attacker’s file immediately.
4.  The DLL is loaded into memory, and its entry point (`DllMain`) executes.
5.  The attacker’s code is now running inside a trusted application.

## Why This is Dangerous

DLL Hijacking is a favorite among attackers because it is incredibly stealthy. 

*   **No Suspicious APIs:** Unlike injection, hijacking doesn't require calling "loud" functions like `CreateRemoteThread` or `WriteProcessMemory`. The application does all the work for the attacker.
*   **Inherited Trust:** Because the malicious code is running inside a legitimate process, it inherits that process's permissions. If the application has network access or admin rights, the attacker does too.
*   **Signed Binaries:** Attackers often use a technique called **DLL Side-loading**. They take a perfectly legitimate, digitally signed EXE (like an old version of an antivirus or a system tool) and bundle it with a malicious DLL. Because the EXE is "trusted" by the OS, it often bypasses security scans, even though it is carrying a hidden payload.

## Real-World Relevance

This isn't just a theoretical trick. DLL hijacking is commonly used in:
*   **Persistence:** Malware can place a hijacked DLL in a folder of an application that starts every time the computer boots up.
*   **Portable Apps:** Software that runs from a USB drive often loads DLLs from its own folder, making it a prime target for this technique.
*   **Privilege Escalation:** If an installer or a system service (running as SYSTEM) loads a DLL insecurely, an attacker can gain full control of the machine.

## Closing Thoughts

DLL Hijacking is a classic example of an "Abuse of Design." The system isn't broken; it is simply working exactly as it was programmed to. By understanding the search order and the way Windows identifies libraries, we can better appreciate why specifying absolute paths and verifying file signatures is a critical part of secure software development.

In our next post, we will wrap up this series by looking at how to **detect and prevent** these types of attacks, from both a developer's and a defender's perspective.

***

***

# DLL Proxying: The Stealthy Middleman

When an attacker successfully hijacks a DLL or performs a side-loading attack, they face a significant hurdle. Modern applications are complex; they might expect hundreds of different functions from a single library. If an attacker replaces `version.dll` with a malicious version that only contains a "Hello World" payload, the application will attempt to call a legitimate function, find that it is missing, and crash instantly.

A crash is "noisy." It alerts the user and the system administrator that something is wrong. To stay hidden, an attacker needs the application to keep running as if nothing happened. This is where **DLL Proxying** comes into play.

## The Core Concept: The Functional Mask

DLL Proxying is a technique where a malicious DLL acts as a middleman. It sits between the application and the original, legitimate DLL. Its job is twofold:
1.  Execute the attacker’s malicious code.
2.  Forward all legitimate function calls to the real library so the application never notices a difference.

In this scenario, the attacker doesn’t just replace the library; they **mimic** it.

## How the Proxy Flow Works

In a standard hijacking scenario, the flow is broken. In a proxying scenario, the flow is preserved through a "forwarding" mechanism.

### The Problem (Simple Hijacking)
*   **App** calls `GetFileVersion()`.
*   **App** loads `malicious_version.dll`.
*   **Malicious DLL** doesn't have `GetFileVersion()`.
*   **Result:** Application Crash.

### The Solution (Proxying)
*   **App** calls `GetFileVersion()`.
*   **App** loads `malicious_version.dll`.
*   **Malicious DLL** catches the call and immediately runs the attacker's payload.
*   **Malicious DLL** then loads the *real* version of the library (often renamed to something like `real_version.dll`).
*   **Malicious DLL** forwards the call for `GetFileVersion()` to the real library.
*   **Real DLL** returns the correct data.
*   **Result:** The attacker's code has run, and the application continues to function perfectly.

## The Visual Mental Model

At runtime, the architecture looks like a chain:

<img width="600" height="500" alt="image" src="https://github.com/user-attachments/assets/259f93c6-2e79-4bad-a58b-e3c96c324577" />



The "Malicious Proxy" is essentially wearing the original DLL as a mask. It presents the correct face to the application while hiding its true nature behind the scenes.

## Why This is the "Gold Standard" for Stealth

DLL Proxying is preferred by advanced persistent threats (APTs) and sophisticated malware for three main reasons:

1.  **Zero Visibility:** Because the application doesn't crash, the user has no reason to investigate. The computer feels "healthy."
2.  **Persistence:** The proxy DLL stays on the disk. Every time the user opens the associated application, the malicious code runs again, and the application continues to work normally.
3.  **Inherited Context:** Just like in injection, the proxy code runs within the memory space of the trusted application, inheriting its network access, file permissions, and digital trust.

## Implementing the Proxy (Conceptually)

From a developer’s perspective, creating a proxy DLL involves a few key steps:

*   **Step 1: The Payload.** Inside the `DllMain` function (the entry point), the attacker places the code they want to run—whether that’s a reverse shell, a keylogger, or a credential stealer.
*   **Step 2: The Load.** The malicious DLL is programmed to find and load the original, renamed DLL using `LoadLibrary`.
*   **Step 3: The Export Forwarding.** This is the technical "magic." The attacker uses linker directives or manual function pointers to ensure that every function the app expects is "exported" by the proxy and redirected to the real DLL.

## Hijacking vs. Side-loading vs. Proxying

It is easy to get these terms confused, so let’s look at how they differ in behavior:

| Technique | Behavior |
| :--- | :--- |
| **Hijacking** | Replacing a legitimate DLL in the search path. |
| **Side-loading** | Bundling a legitimate EXE with a malicious DLL. |
| **Proxying** | Replacing a DLL *and* forwarding calls to the original to prevent crashes. |

## Final Thoughts

DLL Proxying represents the peak of DLL-based attacks. It turns a destructive action (replacing a file) into a constructive one (acting as a bridge). By ensuring that the "victim" application remains stable, the attacker can remain embedded in a system for months or even years without detection.

Understanding these layers—from the basic executable to the complex proxy—is essential for any defender. It teaches us that "it works" does not always mean "it's safe." 

In our final post of this series, we will look at **Defense and Detection**: how can we actually stop these attacks from happening?

***

**Vishwas S Adhikari**  
*Espress0*

