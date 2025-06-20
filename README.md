# SysWhisper-3

SysWhispers3 helps with evasion by generating header/ASM files that implants can use to make direct system calls, bypassing user-mode hooks.

## Official Discord Channel

Come hang out on Discord!

[![Inceptor Server](https://discordapp.com/api/guilds/1155753953108164628/widget.png?style=banner3)](https://discord.gg/f6w6dwZq)

## Overview

Security products, such as AVs and EDRs, often place hooks in user-mode API functions to analyze a program's execution flow and detect potentially malicious activities. SysWhispers2 was designed to generate header/ASM pairs for kernel system calls, allowing direct invocation from C/C++ and bypassing these user-land hooks.

SysWhispers3 builds upon this foundation, integrating new features to address detection patterns and behaviors that emerged with the original SysWhispers2, offering enhanced evasion capabilities.

## Why SysWhispers3 (and not a PR to SysWhispers2)?

The decision to create SysWhispers3 as a standalone version was driven by several factors:

*   **Inceptor Integration:** SysWhispers3 is the "fork" used by [Inceptor][1] and includes utility classes specific to Inceptor's needs, which are not relevant to the original tool.
*   **Compiler Focus:** While SysWhispers2 is moving towards NASM compilation (for gcc/mingw), SysWhispers3 is specifically designed and tested for MSVC, as [Inceptor][1] will remain a Windows-only framework for the foreseeable future.
*   **Experimental Features:** SysWhispers3 contains features (like groundwork for egg-hunting, implemented in Inceptor) that are experimental or partially implemented and would not be suitable for inclusion in the original tool's stable releases.

## Key Features & Differences from SysWhispers2

SysWhispers3 retains the core functionality of SysWhispers2 but introduces several enhancements:

*   **x86/WoW64 Support:** Enables direct syscalls for 32-bit applications, including those running under WoW64 on 64-bit systems.
*   **Syscall Instruction EGG Replacement:** Supports replacing the `syscall` instruction with a placeholder (EGG) to be dynamically resolved at runtime, evading static signatures. (Note: The egg-hunting *resolver* logic is part of [Inceptor][1]).
*   **Direct Jumps to Syscalls:** Implements direct jumps to syscall instructions in both x86 and x64 modes, which can help bypass certain RIP-based detections.
*   **Randomized Jumps to Syscalls:** Extends the direct jump technique by jumping to one of several `syscall` instructions, an idea borrowed from [@ElephantSeal](https://twitter.com/ElephantSe4l/status/1488464546746540042).



## Usage
C:\> python syswhispers.py -h

usage: syswhispers.py [-h] [-p PRESET] [-a {x86,x64}] [-c {msvc,mingw,all}]
                      [-m {embedded,egg_hunter,jumper,jumper_randomized}]
                      [-f FUNCTIONS] -o OUT_FILE [--int2eh] [--wow64] [-v] [-d]

SysWhispers3 - SysWhispers on steroids

optional arguments:

   -h, --help            show this help message and exit
   
  -p PRESET, --preset PRESET
                        Preset ("all", "common")
                        
  -a {x86,x64}, --arch {x86,x64}
                        Architecture
                        
  -c {msvc,mingw,all}, --compiler {msvc,mingw,all}
                        Compiler (msvc is primary, mingw is community supported)
                        
  -m {embedded,egg_hunter,jumper,jumper_randomized}, --method {embedded,egg_hunter,jumper,jumper_randomized}
                        Syscall recovery method
                        
  -f FUNCTIONS, --functions FUNCTIONS
                        Comma-separated functions
                        
  -o OUT_FILE, --out-file OUT_FILE
                        Output basename (w/o extension)
                        
  --int2eh              Use the old `int 2eh` instruction in place of `syscall` (x86 only)
  
  --wow64               Use Wow64 to run x86 on x64 (only usable with x86 architecture)
  
  -v, --verbose         Enable debug output
  
  -d, --debug           Enable syscall debug (insert software breakpoint)



## Standard SysWhispers, 32-bit mode using direct jumps.
    python .\syswhispers.py --preset all -o syscalls_x86_jumper --arch x86 -m jumper

## SysWhispers for specific functions in 32-bit mode targeting WoW64.
    python .\syswhispers.py --functions NtProtectVirtualMemory,NtWriteVirtualMemory -o syscalls_wow64_mem --arch x86 --wow64

## Egg-Hunting SysWhispers (common functions), for dynamic syscall instruction replacement.
    python .\syswhispers.py --preset common -o syscalls_egg_hunter -m egg_hunter

## Jumping Randomized SysWhispers (all functions) for MinGW compiler.
    python .\syswhispers.py --preset all -o syscalls_all_jump_random_mingw -m jumper_randomized -c mingw


## Standard SysWhispers, 32-bit mode using direct jumps.
    python .\syswhispers.py --preset all -o syscalls_x86_jumper --arch x86 -m jumper

## SysWhispers for specific functions in 32-bit mode targeting WoW64.
    python .\syswhispers.py --functions NtProtectVirtualMemory,NtWriteVirtualMemory -o syscalls_wow64_mem --arch x86 --wow64

## Egg-Hunting SysWhispers (common functions), for dynamic syscall instruction replacement.
    python .\syswhispers.py --preset common -o syscalls_egg_hunter -m egg_hunter

## Jumping Randomized SysWhispers (all functions) for MinGW compiler.
    python .\syswhispers.py --preset all -o syscalls_all_jump_random_mingw -m jumper_randomized -c mingw


                      
## SysWhispers3: Why call the kernel when you can whisper?


Common functions selected.

Complete! Files written to:
        temp\syscalls_common.h
        temp\syscalls_common.c
        temp\syscalls_common.asm  (or temp\syscalls_common_asm.asm if MinGW)
Press any key to continue...


## Caveats and Limitations

- Egg-Hunter Implementation: The core egg_hunter syscall generation mode is in this tool. However, the actual run-time EGG resolver logic is implemented within Inceptor, not in SysWhispers3 directly.

- Graphics Subsystem Syscalls: System calls from win32k.sys (graphical subsystem) are not supported.

- Tested Environment: Primarily tested on Visual Studio 2019/2022 with Windows 10 SDK.

### Compiler Support:

- MSVC: Primary supported compiler.

- MinGW: Supported via the -c mingw flag and Makefiles are provided. However, it might not be as extensively tested as MSVC.

- NASM/Other GCC: Support for NASM or other GCC/Clang setups on Windows is not guaranteed.

## Troubleshooting

- General (from SysWhispers2)
  
   Type Redefinition Errors: A project may fail to compile if typedefs in syscalls.h are already defined elsewhere.

- Ensure you only include necessary functions (e.g., --preset all is rarely needed).

- If a typedef is already defined in another header, consider removing it from syscalls.h.

- SysWhispers3 Specific
   Verbose Output: Use the -v or --verbose flag for detailed output during code generation, which can help identify issues.

- Debugging Syscalls: Use the -d or --debug flag to insert a software breakpoint (int3) into the syscall stub, facilitating debugging in tools like WinDbg.

- error A2084:constant value too large (MASM): This error during ASM compilation usually indicates that the syscall numbers have changed (e.g., due to a Windows update) since 
  the function list was last updated or cached. Regenerate the stubs with SysWhispers3, ensuring it fetches the latest syscall numbers.


## Credits
  SysWhispers2
  
   Developed by @Jackson_T and @modexpblog, building upon the work of:
   - @FoxHex0ne: For cataloguing function prototypes and typedefs.

@PetrBenes, NTInternals.net team, and MSDN: For additional prototypes and typedefs.

@Cn33liz: For the initial Dumpert POC.

### SysWhispers2 (x86/WOW64 contributions)
- @rooster: For creating a sample x86/WOW64 compatible fork.

### SysWhispers3 Influences & Ideas
- @ElephantSe4l: For the idea of randomizing jumps to syscalls.

- @S4ntiagoP: For the incredible work on nanodump, which provided inspiration.
