# BitLocker Hardening — Windows 11

> English version available here: [README.en.md](./README.en.md)

## Contexte

Ce dépôt documente une démarche de durcissement de BitLocker sur Windows 11 dans un contexte SOC.

Cette documentation fait suite à la publication d'un article concernant **BitUnlocker**, une attaque de type *downgrade attack* visant certaines configurations BitLocker sur Windows 11.

Source principale :

- CyberSecurity News — BitUnlocker Downgrade Attack on Windows 11  
  https://cybersecuritynews.com/bitunlocker-downgrade-attack-on-windows-11/

## Objectif du dépôt

L'objectif est de fournir une note claire, réutilisable et documentée pour :

- vérifier l'état de BitLocker
- comprendre les limites d'une configuration BitLocker en mode TPM-only
- renforcer la protection contre les attaques avec accès physique
- documenter les mesures de durcissement appliquées sur un poste Windows 11
- produire une trace exploitable dans une logique SOC et Blue Team

## Résumé du risque

BitLocker protège les données au repos. Cependant, certaines attaques physiques peuvent cibler la chaîne de démarrage afin de tenter de contourner ou d'affaiblir la protection.

Le risque est plus important lorsque BitLocker est configuré en mode **TPM-only**.

Dans ce mode, le déverrouillage du volume système peut être effectué automatiquement au démarrage si la chaîne de boot est considérée comme valide.

Une configuration plus robuste consiste à utiliser :

```text
BitLocker + TPM + PIN
```

Avec cette approche, un PIN doit être saisi avant le démarrage complet de Windows. Cela réduit fortement le risque lié à un accès physique non autorisé.

## Menace concernée

Cette documentation cible principalement les scénarios suivants :

- accès physique non autorisé à la machine
- tentative de boot externe via USB ou PXE
- manipulation de la chaîne de démarrage
- downgrade attack contre certains composants de boot
- configuration BitLocker trop permissive en TPM-only

## Vérifier l'état de BitLocker

Commande PowerShell :

```powershell
Get-BitLockerVolume
```

Commande alternative :

```cmd
manage-bde -status
```

## Vérifier les protecteurs BitLocker

Commande :

```cmd
manage-bde -protectors -get C:
```

Points à contrôler :

- présence d'un protecteur TPM
- présence d'une clé de récupération
- présence ou absence d'un PIN au démarrage
- type de protecteur utilisé pour le lecteur système

## Sauvegarder la clé de récupération

Avant toute modification de la configuration BitLocker, il est indispensable de vérifier que la clé de récupération est sauvegardée dans un emplacement sûr.

Exemples d'emplacements possibles :

- compte Microsoft
- coffre-fort de mots de passe
- support externe sécurisé
- export chiffré hors de la machine protégée
- solution d'administration d'entreprise si applicable

Ne jamais stocker la clé de récupération BitLocker dans un dossier public.

## Vérifier Secure Boot

Commande PowerShell :

```powershell
Confirm-SecureBootUEFI
```

Résultat attendu :

```text
True
```

Si le résultat est `False`, Secure Boot doit être vérifié et réactivé depuis l'UEFI ou le BIOS de la machine, si le matériel le permet.

## Activer TPM + PIN

La mesure recommandée consiste à éviter une configuration BitLocker en TPM-only lorsque la machine contient des données sensibles ou peut être exposée à un risque d'accès physique.

Commande possible en invite de commandes administrateurice :

```cmd
manage-bde -protectors -add C: -TPMAndPIN
```

Avant d'exécuter cette commande, il faut vérifier que la stratégie locale autorise l'utilisation d'un PIN au démarrage.

## Configuration via stratégie locale

Ouvrir l'éditeur de stratégie locale :

```text
gpedit.msc
```

Chemin :

```text
Configuration ordinateur
→ Modèles d'administration
→ Composants Windows
→ Chiffrement de lecteur BitLocker
→ Lecteurs du système d'exploitation
```

Paramètre à configurer :

```text
Exiger une authentification supplémentaire au démarrage
```

Recommandation :

```text
Activer l'utilisation d'un PIN de démarrage avec TPM
```

## Durcissement BIOS et UEFI

Mesures recommandées :

- activer Secure Boot
- désactiver le boot USB si non nécessaire
- désactiver le boot réseau ou PXE si non nécessaire
- verrouiller l'ordre de boot sur Windows Boot Manager
- ajouter un mot de passe UEFI ou BIOS
- mettre à jour le firmware de la machine
- éviter de laisser la machine sans surveillance lorsqu'elle est allumée ou en veille

## Mise à jour du système

Actions recommandées :

- installer les dernières mises à jour Windows Update
- installer les mises à jour firmware proposées par le constructeur
- vérifier les mises à jour liées à Secure Boot
- vérifier les mises à jour liées à BitLocker et WinRE
- contrôler régulièrement l'état de sécurité du poste

## Checklist rapide

```text
[ ] Clé de récupération BitLocker sauvegardée
[ ] BitLocker activé sur le lecteur système
[ ] Protecteurs BitLocker vérifiés
[ ] Secure Boot activé
[ ] Mode TPM-only identifié ou écarté
[ ] TPM + PIN activé si nécessaire
[ ] Boot USB limité ou désactivé
[ ] Boot PXE désactivé si non nécessaire
[ ] Ordre de boot verrouillé
[ ] Firmware à jour
[ ] Windows Update à jour
[ ] Documentation locale créée
```

## Note

Cette documentation ne constitue pas une information officielle Microsoft, CIS, ANSSI ou NIST.

Elle constitue une note locale de durcissement et de qualification du risque, utile pour :

- documenter l'état de sécurité d'un poste Windows 11
- mieux comprendre les limites du mode BitLocker TPM-only
- renforcer la protection contre les attaques physiques
- produire une trace exploitable dans un contexte SOC

## Limites

Cette documentation ne remplace pas :

- une politique de sécurité d'entreprise
- un audit complet de configuration Windows
- une analyse forensique
- une revue officielle de conformité
- une validation par un.e administrateurice système ou sécurité

BitLocker protège les données au repos. Il ne protège pas complètement une machine déjà déverrouillée, compromise ou laissée sans surveillance avec une session active.

## Conclusion

BitLocker reste une protection importante pour Windows 11.

Pour un poste sensible, la configuration recommandée est :

```text
BitLocker + TPM + PIN
Secure Boot activé
Boot USB/PXE limité
Firmware à jour
Clé de récupération sauvegardée
```

Le point central est d'éviter de dépendre uniquement d'une configuration TPM-only lorsque le risque d'accès physique doit être pris en compte.

## Références

- CyberSecurity News — BitUnlocker Downgrade Attack on Windows 11  
  https://cybersecuritynews.com/bitunlocker-downgrade-attack-on-windows-11/

- Intrinsec — Contournement BitLocker : la réalité des downgrade attacks  
  https://www.intrinsec.com/en/contournement-bitlocker-la-realite-des-downgrade-attacks/

## Auteurice

Documentation rédigée par **PotiteBulle** dans une logique de durcissement Windows, de documentation de pratique SOC.
