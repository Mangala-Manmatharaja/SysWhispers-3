# SysWhispers3

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

For a more detailed explanation of these features, refer to the blog post: [SysWhispers is dead, long live SysWhispers!][2]

## Installation

```bash
git clone https://github.com/klezVirus/SysWhispers3.git
cd SysWhispers3
python ./syswhispers.py --help
