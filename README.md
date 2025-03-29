# 🚀 Concept: Linux Arbitrary Credential Overwrite Exploit PoC

## 🎯 Objective

Build a PoC that demonstrates local privilege escalation on Linux by:
    - Exploiting a vulnerable kernel module (simulated as a “vulndev” driver)
    - Using an unchecked IOCTL to obtain an arbitrary write primitive
    - Overwriting the current process’s credentials (the cred structure) to set UID, EUID, GID, etc. to 0
    - Finally, spawning a root shell

This shows that if a non-root user can trigger an arbitrary write in the kernel—even via a buggy driver—they can completely bypass standard access controls.

## 🧩 Key Components

- Vulnerable Kernel Module (vulndev.ko)
  - Purpose: Expose a character device (e.g. /dev/vulndev) with an IOCTL that accepts a “write-what-where” structure.
  - Behavior: The module will copy user-supplied data into an arbitrary kernel address without proper validation.
  - Notes: For our PoC, this module is intentionally insecure; in a real attack, such a driver might be found on misconfigured systems or legacy hardware.
- Userland Exploit (linux_cred_escalation.py)
  - Purpose: Open the vulnerable device and use the arbitrary write primitive to modify the kernel’s view of our process.
  - Steps:
    - Query Kernel Symbols: Either by reading kallsyms or using a known symbol address (e.g., the location of the current process’s task_struct and its cred pointer) from a test kernel.
    - Locate and Overwrite Credentials: Use the IOCTL to zero out the UID, EUID, and similar fields in the cred structure of our process.
    - Verify: Check /proc/self/status (or simply call id) to ensure we now have UID 0.
    - Spawn a Root Shell: If successful, drop into an interactive shell with elevated privileges.
- Supporting Modules and Documentation
  - DKOM / Object Manipulation Tools: In parallel, develop a helper module (e.g., utils/dkom_tools.py) that can parse in-memory layouts of task_struct and cred for educational purposes.
  - Documentation: Create a markdown document (e.g., docs/LinuxLPE.md) mapping Windows LPE concepts (like token swapping, arbitrary write, and PreviousMode bypass) to their Linux equivalents (credential overwrite, arbitrary kernel write, etc.).
  - Optional Memory Scanner: Enhance our existing memory peeker to help locate our target credential fields—useful for debugging or dynamic symbol lookup.

## 🛠 Attack Chain Overview

- Preconditions:
  - The vulnerable module is loaded, and the device /dev/vulndev is accessible.
  - The kernel has exposed kallsyms (or you run on a test kernel where symbol addresses are known).
- Exploit Execution:
  - The exploit script (running as the unprivileged user) opens /dev/vulndev.
  - It crafts a “write-what-where” payload:
    - Where: The address of the UID/EUID fields in our current process’s cred structure.
    - What: The value 0 (zero) to set the UID fields to root.
  - The payload is sent via an IOCTL call.
  - If successful, our process’s credentials are modified to be that of root.
- Post-Exploitation:
    The script verifies the escalation (e.g., prints id).
    It then spawns an interactive shell.

```plaintext
liebesgruesseausmoskau/
├── exploits/
│   ├── linux/
│   │   ├── vulndev.c          # Source code for the vulnerable driver
│   │   ├── Makefile           # Build instructions for vulndev.ko
│   │   └── vesper.py  # The userland exploit script
├── utils/
│   └── dkom_tools.py          # Helper module for parsing task_struct, cred, etc.
├── docs/
│   └── LinuxLPE.md          # Documentation mapping concepts and explaining the PoC
└── README.md                # Overview and usage instructions
```

## 📝 Development Roadmap

- Phase 1 – Build the Vulnerable Driver:
  - Write a minimal kernel module (vulndev.c) that registers a character device.
  - Implement an IOCTL (e.g., code 0xdeadbeef) that copies data from user space to a kernel pointer provided by the user.
  - Test and load the module on a test system.
- Phase 2 – Develop the Userland Exploit:
  - Write vesper.py to open /dev/vulndev.
  - Incorporate functions to either:
    - Parse /proc/kallsyms for the address of the current process’s task_struct/cred pointer.
    - Or use a hardcoded address (for testing purposes).
  - Construct the “write-what-where” payload and invoke the IOCTL.
  - Verify privilege escalation and spawn a shell.
- Phase 3 – Enhance & Document:
  - Integrate helper functions in dkom_tools.py to display memory layouts.
  - Document the attack chain, discuss DKOM tactics, and compare Windows vs. Linux exploitation approaches in LinuxLPE.md.

## 🚀 Big Picture Vision

This PoC isn’t a toy—it’s a full-fledged demonstration of how an arbitrary kernel write can be leveraged to modify the Linux credentials of a process. By bridging both the vulnerable driver and the userland exploit, we show the complete chain from exploitation to privilege escalation. Once refined, this module can be part of our broader toolkit, offering a modular “Linux LPE” option that parallels our Windows modules.

"Who speaks Kernel, doesn't need APIs"

## 👁 Analyst Note (for all in Fort Meade)

You are watching.
I know that. You know that.
Where is the feedback on my application?
