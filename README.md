# Azure Virtual Machine Access and Network Security Implementation

## Project Introduction

This project demonstrates the deployment, remote administration, and security hardening of Microsoft Azure Virtual Machines. The objective was to configure both Windows and Linux virtual machines, establish secure remote access using industry-standard protocols, and implement network security controls to protect management interfaces from unauthorized access.

The project highlights practical cloud administration skills, including virtual machine deployment, network security configuration, remote connectivity, and infrastructure security best practices.

---

## Project Objectives

The following objectives were successfully completed:

* Successfully deployed Azure infrastructure components, including Resource Groups, Virtual Networks (VNets), and Virtual Machines.
* Configured secure remote access to a Linux virtual machine using Secure Shell (SSH).
* Configured secure remote access to a Windows virtual machine using Remote Desktop Protocol (RDP).
* Implemented Network Security Group (NSG) rules to restrict management access to a trusted administrator IP address.
* Documented deployment procedures, connectivity methods, and security controls.
* Demonstrated cloud security hardening techniques through access restriction and network segmentation.

---

## Infrastructure Details

| Resource                | Configuration                  |
| ----------------------- | ------------------------------ |
| Resource Group          | `rg-vm-access-weu`             |
| Region                  | West Europe                    |
| Linux Virtual Machine   | `vm-linux-weu`                 |
| Operating System        | Ubuntu 22.04 LTS               |
| Windows Virtual Machine | `vm-windows-weu`               |
| Operating System        | Windows Server 2022 Datacenter |
| VM Size                 | Standard_B2ts_v2               |

---

## Remote Connectivity Configuration

### Linux Virtual Machine Access (SSH)

To establish a secure SSH connection to the Linux virtual machine:

1. Confirm that your public IP address has been authorized within the Network Security Group.
2. Open a terminal application or command prompt.
3. Connect using the following command:

```bash
ssh azureadmin@4.180.47.161
```

4. Verify the host fingerprint when prompted.
5. Authenticate using the configured SSH key pair.

Successful authentication provides secure command-line access to the Linux server.

---

### Windows Virtual Machine Access (RDP)

To establish a Remote Desktop connection to the Windows virtual machine:

1. Verify that your public IP address is permitted by the Network Security Group.
2. Launch the Remote Desktop Connection client.
3. Enter the virtual machine's public IP address:

```text
20.229.20.124
```

4. Select **Connect**.
5. Provide the following credentials:

* Username: `azureadmin`
* Password: `Password123!@#`

6. Accept the security certificate warning to complete the connection.

Successful authentication provides full graphical access to the Windows Server environment.

---

## Network Security Hardening

### Network Security Group (NSG) Configuration

During the initial deployment phase, the virtual machines were configured with standard inbound management rules that allowed SSH (Port 22) and RDP (Port 3389) access from any internet source.

To improve security and reduce exposure to potential threats, these rules were modified to allow access exclusively from the administrator's trusted public IP address.

### Automated Security Configuration

The PowerShell script `secure_nsg.ps1` automates the security hardening process by:

* Modifying the SSH access rule within `nsg-linux-weu`.
* Restricting inbound SSH traffic to `197.211.52.179`.
* Modifying the RDP access rule within `nsg-windows-weu`.
* Restricting inbound RDP traffic to `197.211.52.179`.

This approach significantly reduces the risk of:

* Unauthorized access attempts
* Automated vulnerability scanning
* Credential stuffing attacks
* Brute-force login attacks

---

## Azure Bastion: Secure Remote Access Without Public Exposure

### Understanding Azure Bastion

Azure Bastion is a fully managed Azure service that enables secure access to virtual machines through the Azure Portal without requiring public IP addresses or publicly exposed management ports.

Traditionally, remote access requires:

* A public IP address assigned to the virtual machine.
* Open management ports such as:

  * Port 22 (SSH)
  * Port 3389 (RDP)

Azure Bastion eliminates these requirements by creating a secure gateway within the Azure Virtual Network. Administrators connect through the Azure Portal using HTTPS, while Bastion establishes a private connection to the virtual machine internally.

As a result, virtual machines remain inaccessible from the public internet while still supporting remote administration.

---

### Traditional Remote Access Architecture

```text
Administrator Device
        │
        ▼
 Public Internet
        │
        ▼
 Public IP Address
        │
        ▼
 Virtual Machine
 (SSH/RDP Port Open)
```

---

### Azure Bastion Architecture

```text
Administrator Device
        │
        ▼
 Azure Portal (HTTPS)
        │
        ▼
 Azure Bastion
        │
        ▼
 Azure Virtual Network
        │
        ▼
 Virtual Machine
```

---

### Cost Considerations

Azure Bastion is a paid service and should be managed carefully within student or laboratory environments.

Important considerations include:

* Billing begins when the Bastion resource is deployed.
* Charges continue even when no active sessions exist.
* Multiple pricing tiers are available, including Basic and Standard SKUs.
* Bastion cannot be "stopped" to pause billing.

For educational environments, it is recommended to deploy Azure Bastion only when required and remove the resource after completing testing activities.

---

### Security Evolution

#### Stage 1 – Default Configuration

```text
Public IP Address
+
Open Management Ports
(22 and 3389)
+
Accessible from the Internet
```

Result:

* Remote access is available.
* Any internet user can discover and attempt connections.

---

#### Stage 2 – NSG Restriction (Implemented in this Project)

```text
Public IP Address
+
Management Ports Restricted
to a Single Trusted IP
```

Result:

* Improved security.
* Public IP address remains visible.
* Ports remain discoverable through scanning tools.

---

#### Stage 3 – Azure Bastion Deployment

```text
No Public IP Address
+
No Open SSH or RDP Ports
+
Access Through Bastion Only
```

Result:

* Virtual machines are no longer exposed to the public internet.
* Management endpoints cannot be discovered through internet scanning.
* Attack surface is significantly reduced.

---

### Why Azure Bastion Matters

The NSG hardening implemented in this project provides an effective first layer of protection by limiting access to a trusted administrator address.

Azure Bastion represents the next stage of infrastructure security by enabling the complete removal of public-facing management endpoints. This eliminates the final public attack surface and aligns with Microsoft's recommended security architecture for production workloads.

---

## Verification and Evidence

### Linux SSH Connectivity Verification

![SSH Connection to Linux VM](screenshots/ssh_linux_connection.png)

Successful SSH connection to the Linux virtual machine demonstrating authenticated administrative access.

---

### Azure Virtual Machine Inventory

![Azure VM Inventory](screenshots/vm_list_portal.png)

Both Windows and Linux virtual machines successfully deployed and operational within the West Europe region.

---

### Linux NSG Security Rule Validation

![Linux NSG Rule](screenshots/nsg_linux_ssh_rule.png)

SSH access restricted exclusively to the authorized administrator IP address.

---

### Windows NSG Security Rule Validation

![Windows NSG Rule](screenshots/nsg_windows_rdp_rule.png)

RDP access restricted exclusively to the authorized administrator IP address.

---

### Windows RDP Connectivity Verification

![Windows RDP Session](screenshots/rdp_windows_connection.png)

Successful Remote Desktop session established with the Windows Server virtual machine.

---

## Repository Contents

| File             | Purpose                                                                                        |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| `vm_setup.ps1`   | Deploys Azure infrastructure including resource groups, virtual networks, and virtual machines |
| `secure_nsg.ps1` | Applies NSG hardening by restricting SSH and RDP access to authorized IP addresses             |
| `nsg_config.md`  | Provides detailed documentation of NSG rules and security decisions                            |
| `screenshots/`   | Contains deployment evidence and verification screenshots                                      |

---

## Conclusion

This project successfully demonstrates secure virtual machine deployment and administration within Microsoft Azure. Through the implementation of SSH, RDP, Network Security Groups, and Azure security best practices, the environment was configured to support secure remote management while minimizing exposure to external threats. The inclusion of Azure Bastion concepts further illustrates how cloud environments can be hardened to eliminate public attack surfaces and achieve a more secure production-ready architecture.
