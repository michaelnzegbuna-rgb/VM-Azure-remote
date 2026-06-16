# Network Security Group (NSG) Configuration and Security Assessment Report

## Introduction

This document provides a detailed overview of the Network Security Group (NSG) configurations implemented for the virtual machines deployed in this project. Network Security Groups serve as Azure's built-in network firewall, enabling administrators to control inbound and outbound traffic at both the subnet and network interface levels.

The objective of these configurations is to secure remote administration services while minimizing exposure to unauthorized access and internet-based attacks.

---

## Azure Environment Details

| Resource       | Value              |
| -------------- | ------------------ |
| Resource Group | `rg-vm-access-weu` |
| Region         | West Europe        |

---

## Linux Virtual Machine Security Configuration

### Network Security Group

**NSG Name:** `nsg-linux-weu`

### Inbound Security Rules

| Priority | Rule Name                     | Port | Protocol | Source Address    | Action |
| -------- | ----------------------------- | ---- | -------- | ----------------- | ------ |
| 1000     | default-allow-ssh             | 22   | TCP      | 197.211.52.179    | Allow  |
| 65000    | AllowVnetInBound              | Any  | Any      | VirtualNetwork    | Allow  |
| 65001    | AllowAzureLoadBalancerInBound | Any  | Any      | AzureLoadBalancer | Allow  |
| 65500    | DenyAllInBound                | Any  | Any      | *                 | Deny   |

### Security Justification

The Secure Shell (SSH) service operates on TCP port 22 and is used for remote administration of the Linux virtual machine.

During the initial deployment phase, SSH access was temporarily configured to accept connections from any internet source to facilitate setup and verification. Following successful deployment, the access rule was hardened using the `secure_nsg.ps1` automation script.

The rule now restricts SSH access exclusively to the authorized administrator IP address (`197.211.52.179`). This significantly reduces exposure to:

* Brute-force login attempts
* Automated vulnerability scans
* Unauthorized remote access
* Credential-based attacks

This implementation follows the Principle of Least Privilege by ensuring that only trusted administrative devices can access the server.

---

## Windows Virtual Machine Security Configuration

### Network Security Group

**NSG Name:** `nsg-windows-weu`

### Inbound Security Rules

| Priority | Rule Name                     | Port | Protocol | Source Address    | Action |
| -------- | ----------------------------- | ---- | -------- | ----------------- | ------ |
| 1000     | rdp                           | 3389 | TCP      | 197.211.52.179    | Allow  |
| 65000    | AllowVnetInBound              | Any  | Any      | VirtualNetwork    | Allow  |
| 65001    | AllowAzureLoadBalancerInBound | Any  | Any      | AzureLoadBalancer | Allow  |
| 65500    | DenyAllInBound                | Any  | Any      | *                 | Deny   |

### Security Justification

Remote Desktop Protocol (RDP) operates on TCP port 3389 and provides graphical remote access to Windows systems.

Because RDP services are frequently targeted by attackers, unrestricted internet access presents a significant security risk. To mitigate this threat, the inbound RDP rule was modified to allow connections only from the administrator's trusted public IP address (`197.211.52.179`).

This configuration provides the following security benefits:

* Reduces exposure to internet-based attacks
* Prevents unauthorized login attempts
* Minimizes the likelihood of credential stuffing attacks
* Restricts administrative access to approved locations only

As a result, the Windows virtual machine maintains secure remote accessibility while significantly reducing its attack surface.

---

## Automated Security Enforcement

### NSG Hardening Script

The `secure_nsg.ps1` PowerShell script was developed to automate the security lockdown process for both virtual machines.

Its primary function is to update existing NSG rules and restrict management traffic to the designated administrator IP address.

```powershell
# Restrict SSH access to administrator IP
az network nsg rule update --resource-group rg-vm-access-weu `
    --nsg-name "nsg-linux-weu" `
    --name "default-allow-ssh" `
    --source-address-prefixes 197.211.52.179

# Restrict RDP access to administrator IP
az network nsg rule update --resource-group rg-vm-access-weu `
    --nsg-name "nsg-windows-weu" `
    --name "rdp" `
    --source-address-prefixes 197.211.52.179
```

### Benefits of Automation

Using automation scripts for security enforcement provides several advantages:

* Consistent configuration across environments
* Reduced risk of manual configuration errors
* Faster deployment and remediation processes
* Repeatable and auditable security controls

---

## Security Assessment

The implemented NSG configuration represents a significant improvement over the default deployment model, where management ports are commonly exposed to the entire internet.

### Security Maturity Progression

#### Initial Configuration

```text
Public IP Address
+
SSH/RDP Open to All Sources (*)
```

Risk:

* Highly exposed to scanning and attack attempts.

#### Hardened Configuration (Implemented)

```text
Public IP Address
+
SSH/RDP Restricted to Trusted Administrator IP
```

Benefit:

* Management access remains available while unauthorized access is blocked.

#### Recommended Future Enhancement

```text
No Public IP Address
+
Azure Bastion Access Only
```

Benefit:

* Eliminates publicly exposed management ports entirely.
* Provides a significantly stronger security posture.

---

## Verification Evidence

The following screenshots provide evidence of successful deployment, security configuration, and remote connectivity.

| Screenshot File                          | Purpose                                                            |
| ---------------------------------------- | ------------------------------------------------------------------ |
| `screenshots/vm_list_portal.png`         | Azure Portal view showing both virtual machines in a running state |
| `screenshots/nsg_linux_ssh_rule.png`     | Linux NSG configuration displaying restricted SSH access           |
| `screenshots/nsg_windows_rdp_rule.png`   | Windows NSG configuration displaying restricted RDP access         |
| `screenshots/ssh_linux_connection.png`   | Successful SSH session to the Linux virtual machine                |
| `screenshots/rdp_windows_connection.png` | Successful Remote Desktop session to the Windows virtual machine   |

---

## Conclusion

This NSG implementation demonstrates the practical application of Azure network security controls to protect cloud-hosted virtual machines. By restricting SSH and RDP access to a trusted administrator IP address, the environment adheres to cloud security best practices and significantly reduces exposure to external threats.

The use of automation further strengthens the deployment by ensuring consistent and repeatable security enforcement. Future enhancements, such as Azure Bastion integration and removal of public IP addresses, would provide an additional layer of protection and further reduce the infrastructure's attack surface.
