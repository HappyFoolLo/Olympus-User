# Active Directory Provisioning Script (Mount Olympus Lab Setup)

An automated PowerShell script to quickly build out a structured Active Directory hierarchy, administrator accounts with delegated privileges, and standard user accounts based on a Greek mythology theme (`olympus.net`).

This script is ideal for setting up Active Directory lab environments, testing identity management tooling, or preparing environments for security research and penetration testing scenarios.

## 📌 Features

* **Automated OU Creation**: Programmatically creates an organizational structure (`Mount Olympus` base OU with `Olympians (Admins)` and `Deities (Standard)` sub-OUs).
* **Administrative Account Provisioning**: Populates administrative users with strong, complex passwords (16+ characters).
* **Role & Privilege Delegation**: Dynamically assigns newly created admin accounts to standard AD administrative groups:
  * `Domain Admins`
  * `Server Operators`
  * `Account Operators`
  * `Print Operators`
* **Standard Account Provisioning**: Provisions non-privileged standard domain users with custom UPNs and domain configurations.
* **Idempotent Execution**: Checks for existing OUs and users before attempting creation to prevent runtime errors on repeated executions.

## 🛠️ Prerequisites

* **Windows Server** running Active Directory Domain Services (AD DS) or a domain-joined machine running RSAT.
* **PowerShell 5.1** or **PowerShell 7+**.
* **ActiveDirectory PowerShell Module** installed (`RSAT-AD-PowerShell`).
* Execution with **Domain Admin** privileges.

## 🚀 Usage

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   cd your-repo-name
   ```

2. **Run the Provisioning Script**:
   Launch an elevated PowerShell session (`Run as Administrator`) and execute:
   ```powershell
   .\Provision-OlympusAD.ps1
   ```

## 📁 Active Directory Structure Created

```text
DC=olympus,DC=net
 └── OU=Mount Olympus
      ├── OU=Olympians (Admins)    <-- High privilege & operator accounts
      └── OU=Deities (Standard)    <-- Standard domain accounts
```

## 🔒 Administrative Roles & Group Assignments

| Username | Role Category | Assigned AD Group |
| :--- | :--- | :--- |
| `zeus`, `poseidon`, `hera` | Domain Admin | `Domain Admins` |
| `demeter`, `hephaestus`, `hestia` | Server Admin | `Server Operators` |
| `athena`, `apollo`, `artemis` | Security Admin | `Account Operators` |
| `ares`, `aphrodite`, `hermes` | Helpdesk Admin | `Print Operators` |

## ⚠️ Disclaimer

**Educational / Testing Use Only**: The credentials included in this repository are intended exclusively for lab testing environments. Never use hardcoded secrets in production or commit sensitive production credentials to version control.