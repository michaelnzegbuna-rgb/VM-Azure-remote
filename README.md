# Azure VM Access and Networking Assignment

## Project Overview
This project focuses on the fundamental cloud skill of securely accessing and managing cloud-based virtual machines remotely using Microsoft Azure. I deployed both Windows and Linux virtual machines, configured Network Security Groups (NSGs) for secure remote management, and demonstrated cross-platform connectivity using industry-standard protocols (RDP and SSH).

## Project Goals Achieved
* ✅ Deployed Azure infrastructure including Resource Groups, VNets, and Virtual Machines.
* ✅ Established secure remote connections to a Windows VM using RDP (Remote Desktop Protocol).
* ✅ Established secure remote connections to a Linux VM using SSH (Secure Shell).
* ✅ Implemented security best practices by restricting inbound NSG rules to a specific source IP address.
* ✅ Documented the connectivity procedures and troubleshooting methodologies.

---

## Infrastructure Configuration

- **Resource Group:** `rg-vm-access-weu`
- **Location:** West Europe
- **Linux VM:** `vm-linux-weu` (Ubuntu 22.04 LTS, Standard_B2ts_v2)
- **Windows VM:** `vm-windows-weu` (Windows Server 2022 Datacenter, Standard_B2ts_v2)

---

## Remote Access Procedures

### 1. Connecting to the Linux VM (SSH)
1. Ensure your IP address is whitelisted in the NSG (`nsg-linux-weu`).
2. Open a terminal or command prompt.
3. Use the SSH command with the VM's public IP:
   ```bash
   ssh azureadmin@4.180.47.161
   ```
4. Accept the host key fingerprint if prompted, and authenticate using the generated SSH key.

### 2. Connecting to the Windows VM (RDP)
1. Ensure your IP address is whitelisted in the NSG (`nsg-windows-weu`).
2. Open the **Remote Desktop Connection** client on your local machine.
3. Enter the Windows VM's Public IP: `20.229.20.124`.
4. Click **Connect** and enter the credentials:
   - **Username:** `azureadmin`
   - **Password:** `Password123!@#`
5. Accept the security certificate warning to establish the remote desktop session.

---

## Security Implementation: NSG Configuration

Initially, the Virtual Machines were provisioned with default open ports for RDP (3389) and SSH (22) to the entire internet (`*`). To adhere to cloud security best practices, the Network Security Group rules were updated to restrict inbound management traffic strictly to my local IP address.

### Security Script Execution (`secure_nsg.ps1`)
The provided PowerShell script automates the lockdown of both NSGs:
- Updates `default-allow-ssh` rule on `nsg-linux-weu` to only allow traffic from `197.211.52.179`.
- Updates `default-allow-rdp` rule on `nsg-windows-weu` to only allow traffic from `197.211.52.179`.

This prevents unauthorized scanning and brute-force attacks from malicious actors on the internet.

---

## Evidence

### SSH Connection to Linux VM
![SSH Connection to vm-linux-weu](screenshots/ssh_linux_connection.png)
*Successful SSH session — hostname `vm-linux-weu`, user `azureadmin`, Ubuntu 22.04 LTS*
<img width="1536" height="730" alt="ssh_linux_connection" src="https://github.com/user-attachments/assets/5f8b45d5-5949-427d-b446-e88c6e90c138" />

### VM Inventory (Azure Portal)
![Azure Portal VM List](screenshots/vm_list_portal.png)
*Both VMs running in West Europe on Standard_B2ts_v2*
<img width="1536" height="730" alt="vm_list_portal" src="https://github.com/user-attachments/assets/24d3e97a-7790-4ad3-8fb8-0e8dc40eab1b" />

### Linux NSG — SSH Rule Restricted to Admin IP
![Linux NSG SSH Rule](screenshots/nsg_linux_ssh_rule.png)
*Port 22 restricted to 197.211.52.179 only*
<img width="1536" height="730" alt="nsg_linux_ssh_rule" src="https://github.com/user-attachments/assets/94ce60ad-ed76-40f8-86f3-928cb80b8ef5" />

### Windows NSG — RDP Rule Restricted to Admin IP
![Windows NSG RDP Rule](screenshots/nsg_windows_rdp_rule.png)
*Port 3389 restricted to 197.211.52.179 only*
<img width="1536" height="730" alt="nsg_windows_rdp_rule" src="https://github.com/user-attachments/assets/51569357-6e3c-46e5-b5b0-9aa98fee5a89" />

### RDP Connection to Windows VM
![RDP Connection to vm-windows-weu](screenshots/rdp_windows_connection.png)
*Successful RDP session to `vm-windows-weu` at `20.229.20.124` using credentials `azureadmin`*
<img width="1173" height="672" alt="rdp_windows_connection" src="https://github.com/user-attachments/assets/325c4c68-2757-4208-b379-55c2c165a922" />


---

## Project Files


Here's a clear explanation of that improvement point:

---


### 1. The Concept of Bastion

Right now, the README explains *how to use* Bastion step by step, but a reader who has never heard of it needs to understand *what it is* conceptually before the steps make sense.

The concept to communicate is this: normally, to connect to a VM remotely you need two things — a public IP address on the VM (so the internet can find it) and an open port like 3389 or 22 (so traffic can reach it). Azure Bastion replaces both of those requirements. Instead of your laptop connecting directly to the VM, your browser connects to the Azure Portal over HTTPS, and Bastion — a Microsoft-managed service sitting inside your Virtual Network — makes the final hop to the VM privately. The VM never has to be reachable from the internet at all.

---

### 2. Cost Implications

This matters because Bastion is **not free**, and a reader deploying it without knowing that will get a surprise bill. The documentation should be honest that:

- Bastion charges per deployment hour even when nobody is actively connected
- There are two SKUs (Basic and Standard) with different price points
- For a lab or student environment, the right advice is to **deploy Bastion only when you need it and delete it when you are done** — unlike the VMs themselves, Bastion has no "stopped" state that pauses billing; it bills as long as it exists

Without this explanation, someone might leave Bastion running for a month on a free-tier account and wonder why they have a bill.

---

### 3. How It Enables Removal of Public IPs

This is the most important security point and the one most likely to be glossed over. The documentation should make explicit that Bastion is not just an *alternative* way to connect — it is what **allows you to take the final hardening step** of removing the VM's public IP entirely.

The progression looks like this:

```
Stage 1 — Basic setup
VM has a public IP + port 3389/22 open to everyone (*) 
→ Any internet scanner can attempt to connect

Stage 2 — NSG restriction (what this project does)
VM has a public IP + port 3389/22 open to one specific IP
→ Better, but the port is still visible and the public IP still exists

Stage 3 — Bastion deployed, public IP and ports removed
VM has no public IP + no open management ports
→ There is nothing on the internet for an attacker to target
```

The key insight is that Stage 2 (what the scripts in this project implement) still leaves the VM *findable* on the internet. A port scanner can still see port 3389 is open; it just cannot get past the NSG rule. Stage 3 with Bastion means the VM is genuinely invisible to the internet — no IP, no port, nothing to discover.

---

### Why This Section Matters for the README

Without this explanation, a reader might look at the Bastion deployment steps and think: *"this seems like extra work just to connect differently — why bother?"* The documentation section answers that question. It turns a list of steps into a coherent security argument: Bastion is not just a convenience feature, it is the tool that lets you eliminate the last remaining public attack surface on your VMs.

| File | Description |
|------|-------------|
| `vm_setup.ps1` | Provisions the resource group, Linux VM, and Windows VM with open NSG rules for initial setup |
| `secure_nsg.ps1` | Locks down both NSGs to restrict SSH and RDP to admin IP only |
| `nsg_config.md` | Detailed NSG rule documentation and security rationale |
| `screenshots/` | Evidence screenshots of VMs, connections, and NSG configurations |
