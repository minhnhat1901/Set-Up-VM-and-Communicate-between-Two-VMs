# IT Support Project: Virtual Machine Lab Documentation

This documentation covers a step-by-step process to set up and simulate a small IT support environment using VirtualBox and Windows 10 virtual machines. The goal is to create a networked environment where basic IT tasks can be practiced without needing a physical network.

---

## 1) Download Windows ISO

- Visit the [Microsoft Software Download page](https://www.microsoft.com/en-us/software-download/windows10)
- Download the **Media Creation Tool**
- Double-click on that file to start setup
- Go through each step that the program is asking:
  - For **What do you want to do?**, choose: *Create installation media (USB flash drive, DVD, or ISO file) for another PC* → **Next**
  - For **Select language, architecture, and edition**:
    - If you're from the United States or don’t want to change settings, click **Next**
    - To modify settings, uncheck *Recommended options* and select your preferences → **Next**
  - For **Choose which media to use**, select *ISO file* → **Next**
    - Choose a save location and click **Save**
- The tool will begin downloading the ISO

\* ***Note***: It may take some time to download. If it asks about burning to DVD after download, skip this step.

---

## 2) Download and Install VirtualBox and Attach ISO

### Step 1: Install VirtualBox
- Go to the [VirtualBox download page](https://www.virtualbox.org/wiki/Downloads)
- Download the latest version for your Operating System
- Run the installer and follow the setup instructions

### Step 2: Configure and Attach ISO
- Launch VirtualBox
- Click **New** to create a new virtual machine
- Choose a name (e.g., `Windows10-Project`)
- Choose a folder where you want to save the VM.
```
Note:
- Make sure to choose a directory that has enough space for the VM.
- If you both SSD and HDD, choose SSD for better performance.
- Check Skip Unattended Installation to keep track the installation process.
```
- For ISO Image, select the Windows 10 ISO you recently downloaded.
- Assign memory (e.g., **2048 MB** or more). DO NOT exceed 50% of your physical RAM.
- Assign processor cores (e.g., **2 cores**)
- Create a virtual hard disk (e.g., **50 GB** dynamically allocated)
- Go to Settings > System > Acceleration and choose **Hyper-V**.

---

## 3) Install Windows in the Virtual Machine

- Start the VM
- Follow the Windows Setup process:
  - Choose language, time, and keyboard layout
  - When prompted for a product key, click **"I don't have a key"**
  - Choose **Windows 10 Pro** (or another version)
  - Create a local user account (e.g., `Windows10`)

### Optional:
- After installation, go to **Devices > Insert Guest Additions CD Image**
  - Run the installer inside the VM
  - Reboot the VM to enable better screen resolution, drag-and-drop, and more

---

## 4) Simulate a Small Network (2 VMs)

### Step 1: Create and Clone VM
- After setting up the main VM (`Windows10-Project`), clone it to create `Windows10-Client`
- Choose **Full Clone** to make an independent copy

### Step 2: Configure Internal Network
- For both VMs:
  - Go to **Settings > Network > Adapter 1**
  - Set **Attached to**: `Internal Network`
  - Name: `intnet`
  - Click on **Advanced** and change **MAC Address** on one of the VMs to avoid conflicts
  - Click **OK** to save settings

### Step 3: Assign Static IPs
In each VM, go to:
- **Control Panel** > **Network and Sharing Center** > **Change adapter settings**
- Right-click **Ethernet** > **Internet Protocol Version 4 (TCP/IPv4)** > **Properties**
- Select **Use the following IP address** and enter the following:

For Project:
- IP address: `192.168.10.10`
- Subnet: `255.255.255.0`

For Client:
- IP address: `192.168.10.20`
- Subnet: `255.255.255.0`

Test connectivity:
- Open Command Prompt in each VM
  - For Project VM (IP address: `192.168.10.10`), type:
  ```cmd 
  ping 192.168.10.20
  ```
  For Client VM (IP address: `192.168.10.20`), type:
  ```cmd
  ping 192.168.10.10
  ```
- The expected output should show the reply from the other VM's IP address, along with the time taken for the ping.
- If you see ***`Destination host unreachable`*** or ***`Request timed out`***, that means it's failed to communicate at each other. Start troubleshooting by checking the network settings and firewall.

---

## 5) Practice IT Support Scenarios

### A. Create a New User
- Run **Command Prompt as Administrator**:
```cmd
net user testuser Password! /add
```
- Confirm via `Settings > Accounts > Family & Other Users`

### B. Set File Permissions
- Create a folder: `C:\TestShare`
- Right-click > Properties > **Sharing > Share...**
- Add `testuser` and set permission level to **Read/Write**
- Then go to **Security tab > Edit > Add** and add `testuser` in `Enter the object names to select`
- Assign permission levels (e.g., Read-only)

### C. File Sharing Test
- From Client VM:
  - Open File Explorer
  - Type `\\192.168.10.10\TestShare`
  - Authenticate as `testuser`
  - Try opening, editing, or creating files to test permission rules

### D. Simulate Network Issue
- Disable Ethernet adapter from `Control Panel > Network` on Client VM
- Try pinging or browsing to confirm outage
- Re-enable to restore connection

---

## 6) Install Software Without Network

### Method: Use Shared Folder
1. On Host: Create a folder (e.g., `C:\VM_Share`) and place `.exe` installer inside (e.g., Notepad++)
2. In VirtualBox:
   - VM Settings > **Shared Folders > Add Folder**
   - Folder Path: `C:\VM_Share`
   - Folder Name: `Shared`
   - Enable **Auto-mount** and **Make Permanent**
3. Inside VM:
   - Navigate to `Z:\Shared` in File Explorer
   - **Copy** the installer to `C:\Users\[Username]\Downloads`
   - Run the installer from the local copy

---

## ✅ Troubleshooting & Issues Encountered

### 1) **"The computer restarted unexpectedly or encountered an unexpected error..."**

**How to fix it:**
- Press **Shift + F10** to open Command Prompt
- Type `regedit` and press Enter
- In Registry Editor:
  - Go to: `HKEY_LOCAL_MACHINE\SYSTEM\Setup\Status\ChildCompletion`
  - On the right side, double-click `setup.exe`
  - Change the **Value data** from `1` to `3`
  - Click **OK**, then close Registry Editor
- Back in the Command Prompt, type:
  ```
  shutdown -r -t 0
  ```
- Press Enter to reboot

---

### 2) **VirtualBox “E_FAIL (0x80004005)” when starting second VM**

**Cause:** Not enough RAM available to run both VMs simultaneously

**How to fix it:**
- Shut down both VMs
- Open **VirtualBox > Settings** for each VM
- Go to **System > Motherboard**, and reduce RAM to:
  - 2048 MB or 3072 MB per VM if you have 8–16 GB total RAM
- Save and start the VMs again
- Make sure to run only one VM at a time if you have limited RAM

---

### 3) **Ping between VMs not working (even with correct IPs and firewall off)**

**Cause:** Both VMs might have the **same MAC address**, causing network conflict. This happened when you cloned the existing VM.

**How to fix it:**
- Run `ipconfig /all` in both VMs to check IP Address and MAC addresses
- If they are the same, follow these steps:
  - Change IP address of one VM to avoid conflict.
  - Shut down both VMs
  - In **VirtualBox > Settings > Network > Adapter 1**:
    - Click the circular arrow next to the **MAC Address** field to generate a new address for one of the VMs
  - Optionally, change the **PC name** of one VM to avoid confusion
---

### 4) **Shared folder files can't be run (.exe gives "path does not exist")**

**Cause:** Running `.exe` directly from a VirtualBox shared folder

**How to fix it:**
- Open File Explorer inside the VM
- Copy the file from `Z:\Shared` to a local path like `C:\Users\Windows10\Downloads`
- Run the installer from the copied location

---

### 5) **Both VMs had the same hostname**

**How to fix it:**
- Open **Settings > System > About** inside each VM
- Click **Rename this PC**
- Rename one to something like `ClientVM` or `ProjectVM`
- Reboot to apply changes
