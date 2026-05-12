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

## Check BitLocker status

PowerShell command:

```powershell
Get-BitLockerVolume
```

Alternative command:

```cmd
manage-bde -status
```

## Check BitLocker protectors

Command:

```cmd
manage-bde -protectors -get C:
```

Items to check:

- presence of a TPM protector
- presence of a recovery key
- presence or absence of a startup PIN
- type of protector used for the system drive

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

Possible command from an elevated command prompt:

```cmd
manage-bde -protectors -add C: -TPMAndPIN
```

Before running this command, check that the local policy allows the use of a startup PIN.

## Local policy configuration

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
```

Policy to configure:

```text
Require additional authentication at startup
```

Recommendation:

```text
Enable the use of a startup PIN with TPM
```

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
[ ] TPM + PIN enabled if required
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
