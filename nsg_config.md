# Azure NSG Access Control and Network Security Documentation

## Executive Summary

This document outlines the network security measures implemented for the Azure virtual machines used in this project. The focus of these controls is to protect remote management services by limiting access to authorized users and reducing exposure to external threats.

Azure Network Security Groups (NSGs) are used as traffic-filtering mechanisms that determine which network connections are permitted or denied. These policies help secure cloud resources by controlling communication at both the subnet and network interface levels.

---

# Environment Information

| Component         | Details            |
| ----------------- | ------------------ |
| Resource Group    | `rg-vm-access-weu` |
| Deployment Region | West Europe        |

---

# Linux Virtual Machine Access Controls

### Assigned Network Security Group

`nsg-linux-weu`

### Active Inbound Rules

| Priority | Rule                          | Port | Protocol | Source            | Permission |
| -------- | ----------------------------- | ---- | -------- | ----------------- | ---------- |
| 1000     | default-allow-ssh             | 22   | TCP      | 197.211.52.179    | Allow      |
| 65000    | AllowVnetInBound              | Any  | Any      | VirtualNetwork    | Allow      |
| 65001    | AllowAzureLoadBalancerInBound | Any  | Any      | AzureLoadBalancer | Allow      |
| 65500    | DenyAllInBound                | Any  | Any      | *                 | Deny       |

### Access Control Strategy

SSH connectivity is required to remotely manage the Linux server. During the deployment phase, SSH access was temporarily available from any source to simplify installation and testing activities.

Once deployment validation was completed, the security posture was strengthened by updating the NSG rule to permit SSH traffic only from the administrator's trusted public IP address (`197.211.52.179`).

This restriction helps protect the server from:

* Unauthorized connection attempts
* Password guessing attacks
* Automated reconnaissance tools
* Internet-wide vulnerability scans

By narrowing access to a single trusted source, administrative connectivity remains available while unnecessary exposure is removed.

---

# Windows Virtual Machine Access Controls

### Assigned Network Security Group

`nsg-windows-weu`

### Active Inbound Rules

| Priority | Rule                          | Port | Protocol | Source            | Permission |
| -------- | ----------------------------- | ---- | -------- | ----------------- | ---------- |
| 1000     | rdp                           | 3389 | TCP      | 197.211.52.179    | Allow      |
| 65000    | AllowVnetInBound              | Any  | Any      | VirtualNetwork    | Allow      |
| 65001    | AllowAzureLoadBalancerInBound | Any  | Any      | AzureLoadBalancer | Allow      |
| 65500    | DenyAllInBound                | Any  | Any      | *                 | Deny       |

### Access Control Strategy

Remote Desktop Protocol (RDP) is used to administer the Windows virtual machine through a graphical interface.

Because RDP services are frequently targeted by cyberattacks, unrestricted internet access introduces unnecessary risk. To strengthen security, the default rule was modified so that only traffic originating from the administrator's approved IP address (`197.211.52.179`) is accepted.

This approach provides several advantages:

* Blocks unsolicited connection attempts from unknown sources
* Limits exposure to internet-based threats
* Reduces the likelihood of successful credential attacks
* Ensures administrative access is restricted to trusted locations

As a result, the Windows server remains manageable while maintaining a more secure network posture.

---

# Automated NSG Hardening

## Security Automation Script

To streamline security configuration, a PowerShell script named `secure_nsg.ps1` was created. The script automatically updates the NSG rules associated with both virtual machines and applies source IP restrictions.

```powershell
# Restrict SSH access
az network nsg rule update --resource-group rg-vm-access-weu `
    --nsg-name "nsg-linux-weu" `
    --name "default-allow-ssh" `
    --source-address-prefixes 197.211.52.179

# Restrict RDP access
az network nsg rule update --resource-group rg-vm-access-weu `
    --nsg-name "nsg-windows-weu" `
    --name "rdp" `
    --source-address-prefixes 197.211.52.179
```

## Advantages of Automation

Automating security configuration offers several operational benefits:

* Ensures consistency across deployments
* Reduces manual administration effort
* Minimizes configuration mistakes
* Enables repeatable security practices
* Supports easier auditing and compliance validation

---

# Security Posture Evaluation

The final configuration represents a significant security improvement compared to the default deployment model commonly used for testing environments.

## Phase 1: Initial Deployment

```text
Public IP Address
+
Open SSH/RDP Access from Any Internet Source
```

### Security Impact

* Maximum exposure to internet traffic
* High visibility to automated scanners
* Increased likelihood of attack attempts

---

## Phase 2: Restricted Administrative Access (Implemented)

```text
Public IP Address
+
SSH/RDP Accessible Only from Approved Administrator IP
```

### Security Impact

* Administrative access remains functional
* Unauthorized internet users are blocked
* Attack surface is substantially reduced

---

## Phase 3: Recommended Future State

```text
No Public IP Address
+
Azure Bastion for Remote Management
```

### Security Impact

* Eliminates publicly accessible management ports
* Removes direct internet exposure
* Provides a stronger zero-trust security model

### Why Azure Bastion Matters

Azure Bastion allows administrators to connect to virtual machines through the Azure Portal using HTTPS rather than exposing SSH or RDP directly to the internet.

With Bastion deployed:

* Public IP addresses can be removed from virtual machines
* Ports 22 and 3389 no longer need internet exposure
* Remote access remains available through the Azure Portal
* The overall attack surface is significantly reduced

This represents the most secure architecture for remote VM administration in Azure.

---

# Validation and Supporting Evidence

The following screenshots were collected to verify successful deployment, connectivity testing, and security configuration.

| Evidence File                            | Verification Purpose                                                     |
| ---------------------------------------- | ------------------------------------------------------------------------ |
| `screenshots/vm_list_portal.png`         | Confirms both virtual machines are successfully deployed and operational |
| `screenshots/nsg_linux_ssh_rule.png`     | Shows SSH access restricted to the approved administrator IP             |
| `screenshots/nsg_windows_rdp_rule.png`   | Shows RDP access restricted to the approved administrator IP             |
| `screenshots/ssh_linux_connection.png`   | Demonstrates successful Linux VM access through SSH                      |
| `screenshots/rdp_windows_connection.png` | Demonstrates successful Windows VM access through Remote Desktop         |

---

# Final Remarks

The network security controls implemented in this project demonstrate how Azure Network Security Groups can be used to protect cloud-hosted virtual machines from unnecessary exposure.

By restricting management protocols to a trusted source address and automating policy enforcement through scripting, the environment achieves a stronger security posture while preserving administrative accessibility.

For future improvements, Azure Bastion should be considered as the preferred remote access solution, allowing the complete removal of public-facing management endpoints and providing a more secure cloud architecture.
