# BitLocker Hardening — Windows 11

> French version available here: [README.md](./README.md)

## Context

This repository documents a BitLocker hardening approach for Windows 11 in a SOC context.

This documentation was created after the publication of an article about **BitUnlocker**, a *downgrade attack* targeting certain BitLocker configurations on Windows 11.

Main source:

- CyberSecurity News — BitUnlocker Downgrade Attack on Windows 11  
  https://cybersecuritynews.com/bitunlocker-downgrade-attack-on-windows-11/

## Repository objective

The objective is to provide a clear, reusable, and documented note to:

- check the BitLocker status
- understand the limits of a TPM-only BitLocker configuration
- strengthen protection against physical access attacks
- document hardening measures applied to a Windows 11 workstation
- produce useful documentation in a SOC and Blue Team context

## Risk summary

BitLocker protects data at rest. However, some physical attacks can target the boot chain in an attempt to bypass or weaken the protection.

The risk is higher when BitLocker is configured in **TPM-only** mode.

In this mode, the system volume may be unlocked automatically during startup if the boot chain is considered valid.

A stronger configuration is:

```text
BitLocker + TPM + PIN
```

With this approach, a PIN must be entered before Windows fully starts. This greatly reduces the risk related to unauthorized physical access.

## Relevant threat model

This documentation mainly focuses on the following scenarios:

- unauthorized physical access to the machine
- external boot attempt via USB or PXE
- boot chain manipulation
- downgrade attack against certain boot components
- overly permissive TPM-only BitLocker configuration

## Recommended state

Recommended configuration for a sensitive workstation:

```text
BitLocker enabled
Secure Boot enabled
TPM enabled
Recovery key backed up
TPM + PIN protector enabled
USB boot limited if not required
PXE boot disabled if not required
Firmware up to date
Windows Update up to date
```

## Check BitLocker status

PowerShell command:

```powershell
Get-BitLockerVolume
```

Alternative command:

```cmd
manage-bde -status C:
```

Useful items to check:

```text
Conversion Status
Protection Status
Lock Status
Encryption Method
```

## Check BitLocker protectors

Command:

```cmd
manage-bde -protectors -get C:
```

In a TPM-only configuration, the result may only show:

```text
TPM
Numerical Password
```

The **Numerical Password** is the BitLocker recovery key.

In a hardened configuration, the expected protector is:

```text
TPM and PIN
```

## Back up the recovery key

Before making any change to the BitLocker configuration, it is essential to verify that the recovery key is backed up in a safe location.

Examples of possible storage locations:

- Microsoft account
- password manager
- secured external storage
- encrypted export outside the protected machine
- enterprise administration solution if applicable

Never store the BitLocker recovery key in a public folder.

## Check Secure Boot

PowerShell command:

```powershell
Confirm-SecureBootUEFI
```

Expected result:

```text
True
```

If the result is `False`, Secure Boot should be checked and re-enabled from the machine UEFI or BIOS, if supported by the hardware.

## Enable TPM + PIN

The recommended measure is to avoid a TPM-only BitLocker configuration when the machine contains sensitive data or may be exposed to physical access risk.

### Step 1 — Open Local Group Policy

Open the Local Group Policy Editor:

```text
gpedit.msc
```

Path:

```text
Computer Configuration
→ Administrative Templates
→ Windows Components
→ BitLocker Drive Encryption
→ Operating System Drives
→ Require additional authentication at startup
```

### Step 2 — Enable the policy

Configure the policy as follows:

```text
State : Enabled
```

### Step 3 — Configure TPM options

Recommended configuration:

```text
[ ] Allow BitLocker without a compatible TPM

Configure TPM startup:
→ Allow TPM

Configure TPM startup PIN:
→ Require startup PIN with TPM

Configure TPM startup key:
→ Do not allow startup key with TPM

Configure TPM startup key and PIN:
→ Do not allow startup key and PIN with TPM
```

The option **Allow BitLocker without a compatible TPM** should be unchecked if the machine already has a working TPM.

### Step 4 — Apply the policy

Click:

```text
Apply
OK
```

Then run from an elevated terminal:

```cmd
gpupdate /force
```

### Step 5 — Add the TPM + PIN protector

From an elevated terminal:

```cmd
manage-bde -protectors -add C: -TPMAndPIN
```

Windows will then ask you to define a BitLocker PIN.

Recommendations:

- use a PIN you can remember
- avoid weak values such as 000000 or 123456
- do not store the PIN in a public file
- keep the BitLocker recovery key available before the first reboot

### Step 6 — Verify the configuration

Command:

```cmd
manage-bde -protectors -get C:
```

The expected result should include a protector of type:

```text
TPM and PIN
```

### Step 7 — Test on reboot

Command:

```cmd
shutdown /r /t 0
```

On reboot, Windows should ask for the BitLocker PIN before fully loading the system.

## BIOS and UEFI hardening

Recommended measures:

- enable Secure Boot
- disable USB boot if not required
- disable network boot or PXE if not required
- lock the boot order to Windows Boot Manager
- add a UEFI or BIOS password
- update the machine firmware
- avoid leaving the machine unattended while it is powered on or sleeping

## System update

Recommended actions:

- install the latest Windows Updates
- install firmware updates provided by the manufacturer
- check updates related to Secure Boot
- check updates related to BitLocker and WinRE
- regularly check the security state of the workstation

## Quick checklist

```text
[ ] BitLocker recovery key backed up
[ ] BitLocker enabled on the system drive
[ ] BitLocker protectors checked
[ ] Secure Boot enabled
[ ] TPM-only mode identified or ruled out
[ ] TPM + PIN policy configured
[ ] TPM + PIN protector added
[ ] Reboot test completed
[ ] USB boot limited or disabled
[ ] PXE boot disabled if not required
[ ] Boot order locked
[ ] Firmware up to date
[ ] Windows Update up to date
[ ] Local documentation created
```

## Note

This documentation is not official information from Microsoft, CIS, ANSSI, or NIST.

It is a local hardening and risk qualification note, useful to:

- document the security state of a Windows 11 workstation
- better understand the limits of BitLocker TPM-only mode
- strengthen protection against physical access attacks
- produce useful documentation in a SOC context

## Limitations

This documentation does not replace:

- an enterprise security policy
- a full Windows configuration audit
- a forensic analysis
- an official compliance review
- validation by a system or security administrator

BitLocker protects data at rest. It does not fully protect a machine that is already unlocked, compromised, or left unattended with an active user session.

## Conclusion

BitLocker remains an important protection for Windows 11.

For a sensitive workstation, the recommended configuration is:

```text
BitLocker + TPM + PIN
Secure Boot enabled
USB/PXE boot limited
Firmware up to date
Recovery key backed up
```

The central point is to avoid relying only on a TPM-only configuration when physical access risk must be taken into account.

## References

- CyberSecurity News — BitUnlocker Downgrade Attack on Windows 11  
  https://cybersecuritynews.com/bitunlocker-downgrade-attack-on-windows-11/

- Intrinsec — Contournement BitLocker : la réalité des downgrade attacks  
  https://www.intrinsec.com/en/contournement-bitlocker-la-realite-des-downgrade-attacks/

## Author

Documentation written by **PotiteBulle** as part of Windows hardening and SOC practice documentation.
