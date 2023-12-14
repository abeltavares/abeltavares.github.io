---
layout: post
title: "WSL 2: Introduction to Linux on Windows"
date: 2023-12-14T10:45:00-07:00
tags: OS
---

Today, we explore Windows Subsystem for Linux 2 (WSL 2), understanding its capabilities while acknowledging its distinctions from a native Linux environment. Additionally, I'll guide you through the step-by-step installation process to integrate Linux into your Windows enviroment.

## Understanding WSL 2: Fusing Windows and Linux

WSL 2 stands as a significant milestone in the collaboration between Windows and Linux, employing a genuine Linux kernel within the Windows environment. Let's discover its strengths and understand where it differs from typical Linux systems.

### Strengths of WSL 2:

- **Optimized Performance:** Leveraging an authentic Linux kernel, WSL 2 elevates the execution speed and efficiency of Linux applications on Windows platforms.
  
- **Enhanced System Call Compatibility:** Expanding support for system calls, ensuring better compatibility with a wide array of Linux applications and tools.
  
- **Seamless Docker Integration:** Facilitating the direct execution of Docker containers within the WSL 2 environment, streamlining development workflows and processes.

### Considerations regarding WSL 2:

- **Absence of Native Graphical Interface:** WSL 2 primarily operates via a command-line interface, limiting functionality for applications reliant on graphical interfaces.
  
- **Restricted Kernel Configuration:** Customization options for kernel configurations are more limited compared to a dedicated Linux setup.
  
- **Hardware Accessibility:** Certain hardware functionalities might not be fully accessible within WSL 2, impacting specific advanced tasks that require low-level hardware interaction.

## Guide to WSL 2 Installation

**Prerequisites:**

- Windows 10 (version 1903 or higher) with Build 18362 or later.
- Enabled virtualization support in BIOS settings.
- Administrator permissions to enable Windows features.

**Step-by-Step Installation Process:**

1. **Activate Virtual Machine Platform:**
   - Open PowerShell as an administrator.
   - Execute: `dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`

2. **Enable WSL 2 Feature:**
   - In PowerShell (admin mode), enter: `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`

3. **Install Linux Kernel Update Package:**
   - Download and install the latest Linux kernel update package from [here](https://aka.ms/wsl2kernel).

4. **Set WSL 2 as Default Version:**
   - Launch PowerShell as an admin.
   - Input: `wsl --set-default-version 2`

5. **Download and Initialize Your Preferred Linux Distribution:**
   - Visit the Microsoft Store.
   - Find and download your chosen Linux distribution (e.g., Ubuntu, Debian).
   - Open the installed Linux from the Start menu and complete the initial setup by creating a username and password.

## Embrace the Versatility of WSL 2

While WSL 2 may not replicate the complete native Linux environment, it serves as a powerful tool bridging Windows and Linux functionalities.By following this installation guide, you're starting an exciting journey that combines the best of both systems, offering diverse opportunities for development and exploration.

Start exploring WSL 2 today – a place where Windows and Linux collaborate to produce innovation and boost productivity! 🌟
