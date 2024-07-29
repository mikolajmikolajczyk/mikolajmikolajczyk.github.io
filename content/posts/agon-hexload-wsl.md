---
title: "[Agon Light] How to Use Hexload Utility on WSL for Agon Computer"
date: 2024-07-28T19:31:24+02:00
draft: false
toc: false
images:
tags: 
  - agon
  - apps
---

[Hexload](https://github.com/envenomator/agon-hexload) utility for Agon is a life saver when developing applications as it removes the need to swap SD card between your PC and Agon constantly to test changes. This is especially useful when emulator is not enough (when you play with GPIOs for example). This blog post will guide you through the necessary steps to set up and use the hexload utility, ensuring smooth communication between your PC and the Agon computer.

## Prerequisites

1. **Ubuntu on WSL**: Install Ubuntu from Microsoft Store if you haven't already.
2. **usbipd**: Download and install usbipd tool from [this link](https://github.com/dorssel/usbipd-win/releases)
3. **agon-hexload**: Clone or download the `agon-hexload` from [repo](https://github.com/envenomator/agon-hexload)

## Step-by-Step Procedure

### Step 1: Prepare hexload tool files

1. Download `hexload.bin` and `hexload.dll`: Get these files [from agon-hexload releases page](https://github.com/envenomator/agon-hexload/releases)
2. Copy Files to Agon SD Card: Insert the SD card into your Agon computer and copy `hexload.bin` and `hexload.dll` to the `mos/` directory on the SD card.

### Step 2: Start hexload on Agon

1. Connect up Agon to one of your PC USB ports. Agon will get powered up and it will be detected by Windows as CH340 serial device.
2. Now if you want to load program directly into 0x040000 you can just type `hexload vdp` - then you just need to type `run` to execute it. If you want to save program as file on sd card you should type `hexload vdp FILENAME`.

### Step 3: Install usbipd tool

1. **Download [usbipd](https://github.com/dorssel/usbipd-win/releases)** and install it

### Step 4: Identify USB Device

1. **List available devices**: Open PowerShell as admin and type `usbipd.exe list` and press Enter. This command lists all devices connected to USB interface.
2. **Find USB-SERIAL CH340**: Look for device named "USB-SERIAL CH340 (COM X)" in the list and note down its BUSID

### Step 5: Attach USB Device to WSL

1. **Bind USB Device**: In PowerShell, execute following command:
```powershell
usbipd bind --busid BUSID
```
Replace BUSID with the actual BUSID noted earlier.
2. **Attach Device to WSL**: Then, run this command:
```powershell
usbipd attach --wsl --busid BUSID
```

### Step 6: Use Hexload Utility on WSL

1. **Navigate to Agon-Hexload Directory**: Open a terminal in Ubuntu on WSL and navigate to the [agon-hexload](https://github.com/envenomator/agon-hexload) directory.
2. **Run send.py Script**: Ensure you have Python 3 installed. You can now use the send.py script to send files to the Agon computer. Run the following command:
```bash
python3 send.py FILENAME
```
Replace `FILENAME` with the name of the file you want to send.


The `usbipd` tool mounts COM port as `/dev/ttyUSB0`, which is used by default by the `send.py` script on Linux. This setup will allow seamless communication between your WSL environment and the Agon computer.

### Final notes

There is one thing that you need to remember about when using this method. When your PC goes to sleep you will need to do `usbipd attach --wsl --busid BUSID` again.

#### Happy coding!