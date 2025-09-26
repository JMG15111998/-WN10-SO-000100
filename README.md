WN10-CC-000327 
# ✍️ Vulnerability Management Lab – WN10-CC-000327

**Title:** PowerShell Transcription Must Be Enabled  
**STIG ID:** WN10-CC-000327  
**Compliance Standards:** DISA STIG, NIST 800-53, HIPAA, PCI DSS  
**Tools Used:** Azure, Windows 10, PowerShell, Tenable.sc / Nessus  
**Lab Type:** Vulnerability Simulation → Detection → Remediation → Verification

---

## 📋 Lab Objective

This lab demonstrates how to detect and remediate a misconfiguration related to **PowerShell Transcription**, a security logging feature that captures command-line input and output for forensic review. Enabling transcription is required for **STIG WN10-CC-000327** compliance.

You will:
- Deploy a Windows 10 VM in Azure  
- Simulate the vulnerability by disabling transcription  
- Scan the system using Tenable STIG audits  
- Remediate using PowerShell  
- Verify the change manually and via a rescan  

---

## 📁 Table of Contents

1. [Azure VM Setup](#azure-vm-setup)  
2. [Vulnerability Simulation](#vulnerability-simulation)  
3. [Tenable Scan Configuration](#tenable-scan-configuration)  
4. [Initial Vulnerability Scan](#initial-vulnerability-scan)  
5. [Remediation via PowerShell](#remediation-via-powershell)  
6. [Post-Remediation Verification](#post-remediation-verification)  
7. [Security Rationale](#security-rationale)  
8. [Appendix: PowerShell Commands](#appendix-powershell-commands)

---

## ☁️ Azure VM Setup

### 🔹 VM Provisioning

| Setting              | Value                          |
|----------------------|----------------------------------|
| VM Name              | `Win10-STIGLab-Transcription`   |
| OS Image             | Windows 10 Pro (Gen 2)           |
| VM Size              | Standard D2s v3                  |
| Resource Group       | `vm-lab-transcription`           |
| Region               | Closest to analyst               |

### 🔹 Credential Requirements

> ⚠️ **Avoid weak credentials** like `labuser / Cyberlab123!`  
> ✅ Use complex, unique passwords stored securely.

### 🔹 Network Security Group (NSG)

| Port | Protocol | Direction | Status  |
|------|----------|-----------|---------|
| 3389 | RDP      | Inbound   | ✅ Allow |
| 5985 | WinRM    | Inbound   | ✅ Allow (optional) |
| *    | All      | Inbound   | ❌ Deny  |

### 🔹 Local Configuration

#### Disable Windows Firewall *(lab only)*  
- Open `wf.msc`  
- Disable Domain, Private, and Public firewalls

#### Enable Remote PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

---

## ⚠️ Vulnerability Simulation

### 🔸 Vulnerability Summary

**PowerShell transcription** logs every command entered and its output, providing critical evidence for security investigations. If disabled, command-line activity can be hidden from logging mechanisms.

### 🔸 Simulate the Misconfiguration

```powershell
# Simulate insecure configuration by disabling transcription
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription" `
  -Name "EnableTranscripting" -Value 0
```

> `0` = Transcription disabled (non-compliant)  
> `1` = Transcription enabled (compliant)

📸 **Screenshot Placeholder:** `Screenshot_01_Transcription_Disabled_PowerShell.png`

---

## 🔍 Tenable Scan Configuration

### 🔸 Scan Template: Advanced Network Scan

#### ✅ Basic Settings
- Name: `STIG Scan – PowerShell Transcription`
- Target: Public IP of the Azure VM

#### ✅ Discovery Settings
- Ping remote host  
- TCP full connect scan  
- SMB / NetBIOS discovery

#### ✅ Assessment Settings
- Authenticated scan with local admin credentials
- Enable:
  - Remote Registry  
  - Admin Shares  
  - Server Service  
  - Thorough checks

#### ✅ Compliance Checks
- Upload and assign audit file:  
  `DISA STIG – Microsoft Windows 10 v3r4.audit`

---

## 🧪 Initial Vulnerability Scan

Once the scan completes in Tenable:

| STIG Control        | WN10-CC-000327             |
|---------------------|----------------------------|
| Finding             | PowerShell Transcription Disabled |
| Status              | ❌ **Fail**                 |
| Detected Value      | `EnableTranscripting = 0`  |
| Required Value      | `EnableTranscripting = 1`  |

📸 **Screenshot Placeholder:** `Screenshot_02_Tenable_Vuln_Finding_BeforeFix.png`

---

## 🛠️ Remediation via PowerShell

### 🔸 Enable PowerShell Transcription Logging

```powershell
# Enable PowerShell Transcription (compliant configuration)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription" `
  -Name "EnableTranscripting" -Value 1 -Type DWord
```

📸 **Screenshot Placeholder:** `Screenshot_03_PowerShell_Remediation_TranscriptionEnabled.png`

> Transcription settings are applied instantly but may require reboot to be picked up by group policies.

---

## 🔁 Post-Remediation Verification

### 🔸 Manual Registry Check

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription" `
  -Name "EnableTranscripting"
```

**Expected Output:**
```
EnableTranscripting : 1
```

### 🔸 Re-run Tenable Compliance Scan

| STIG Control | WN10-CC-000327 |
|--------------|----------------|
| Status       | ✅ **Pass**     |
| Confirmed    | Transcription Enabled |

📸 **Screenshot Placeholder:** `Screenshot_04_Tenable_AfterRemediation_Pass.png`

---

## 🔐 Security Rationale

### Why It Matters

If transcription is disabled:
- Security teams lose visibility into PowerShell-based attacks  
- Malicious commands can go undetected  
- Forensic investigations are incomplete or impossible  

Enabling transcription ensures **audit trails** for every PowerShell session — critical in regulated environments.

### Compliance Mapping

| Standard       | Requirement Description                     |
|----------------|----------------------------------------------|
| **DISA STIG**  | WN10-CC-000327 – Enable PowerShell Transcription |
| **NIST 800-53**| AU-2, AU-6 – Audit Events & Review           |
| **HIPAA**      | §164.312(b) – Audit Controls                 |
| **PCI DSS**    | 10.2.5 – Logging of Administrative Actions   |

---

## 🧼 Post-Lab Cleanup

- ✅ Reboot VM to confirm logging is active  
- 🧹 Delete the lab resource group after validation:

```bash
az group delete --name vm-lab-transcription --yes --no-wait
```

- 🔐 Remove any saved Tenable scan credentials from the lab

---

## 📎 Appendix: PowerShell Commands

| Task                      | PowerShell Command |
|---------------------------|--------------------|
| Simulate vulnerability    | `Set-ItemProperty -Path HKLM:\...\Transcription -Name EnableTranscripting -Value 0` |
| Apply secure config       | `Set-ItemProperty -Path HKLM:\...\Transcription -Name EnableTranscripting -Value 1` |
| Verify setting            | `Get-ItemProperty -Path HKLM:\...\Transcription -Name EnableTranscripting` |
| Enable remote PowerShell  | `Set-ItemProperty -Path HKLM:\...\System -Name LocalAccountTokenFilterPolicy -Value 1` |

---

✅ **Lab Complete**

You've successfully executed the end-to-end vulnerability management cycle for **WN10-CC-000327** using Azure infrastructure, PowerShell remediation, and Tenable STIG scanning.

Explore `/labs/` for more Windows 10 STIG exercises.
