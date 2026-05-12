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

## Clarification importante

Cette documentation ne signifie pas que BitLocker est cassé ou inutilisable en mode TPM-only.

Dans une configuration saine, avec Secure Boot activé, un TPM fonctionnel et une chaîne de démarrage cohérente, BitLocker peut détecter certaines modifications de l’environnement de boot. Par exemple, une modification de l’ordre de démarrage, un boot USB, un boot PXE ou un changement dans l’état mesuré par le TPM peut déclencher une demande de clé de récupération BitLocker.

Le point important est que le mode TPM-only repose fortement sur la confiance accordée à la chaîne de démarrage. Il reste pratique et adapté à de nombreux usages, mais il n’ajoute pas de pré-authentification utilisateur avant le déverrouillage du volume système.

Pour un poste sensible ou exposé à un risque d’accès physique, la configuration TPM + PIN ajoute une couche supplémentaire. Elle oblige la saisie d’un PIN avant le démarrage complet de Windows, ce qui réduit la dépendance à une configuration uniquement basée sur le TPM et l’état du boot.

**Cette documentation doit donc être comprise comme une note de durcissement, pas comme une alerte indiquant que BitLocker serait défaillant.**


## Menace concernée

Cette documentation cible principalement les scénarios suivants :

- accès physique non autorisé à la machine
- tentative de boot externe via USB ou PXE
- manipulation de la chaîne de démarrage
- downgrade attack contre certains composants de boot
- configuration BitLocker trop permissive en TPM-only

## État recommandé

Configuration recommandée pour un poste sensible :

```text
BitLocker activé
Secure Boot activé
TPM activé
Clé de récupération sauvegardée
Protecteur TPM + PIN activé
Boot USB limité si non nécessaire
Boot PXE désactivé si non nécessaire
Firmware à jour
Windows Update à jour
```

## Vérifier l'état de BitLocker

Commande PowerShell :

```powershell
Get-BitLockerVolume
```

Commande alternative :

```cmd
manage-bde -status C:
```

Points utiles à vérifier :

```text
Conversion Status
Protection Status
Lock Status
Encryption Method
```

## Vérifier les protecteurs BitLocker

Commande :

```cmd
manage-bde -protectors -get C:
```

Dans une configuration TPM-only, on peut voir uniquement :

```text
TPM
Mot de passe numérique
```

Le **mot de passe numérique** correspond à la clé de récupération BitLocker.

Dans une configuration renforcée, on cherche à obtenir un protecteur de type :

```text
TPM et PIN
```

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

### Étape 1 — Ouvrir la stratégie locale

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
→ Exiger une authentification supplémentaire au démarrage
```

### Étape 2 — Activer la stratégie

Configurer la stratégie comme suit :

```text
État : Activé
```

### Étape 3 — Configurer les options TPM

Configuration recommandée :

```text
[ ] Autoriser BitLocker sans un module de plateforme sécurisée compatible

Configurer le démarrage du module de plateforme sécurisée :
→ Autoriser le module de plateforme sécurisée

Configurer le code PIN de démarrage du module de plateforme sécurisée :
→ Exiger un code PIN de démarrage avec le module de plateforme sécurisée

Configurer la clé de démarrage du module de plateforme sécurisée :
→ Ne pas autoriser de clé de démarrage avec le module de plateforme sécurisée

Configurer le code PIN et la clé de démarrage du module de plateforme sécurisée :
→ Ne pas autoriser de clé et de code PIN de démarrage avec le module de plateforme sécurisée
```

La case **Autoriser BitLocker sans un module de plateforme sécurisée compatible** doit être décochée si la machine possède déjà un TPM fonctionnel.

### Étape 4 — Appliquer la stratégie

Cliquer sur :

```text
Appliquer
OK
```

Puis lancer dans un terminal administrateur :

```cmd
gpupdate /force
```

### Étape 5 — Ajouter le protecteur TPM + PIN

Dans un terminal administrateur :

```cmd
manage-bde -protectors -add C: -TPMAndPIN
```

Windows demande alors de définir un PIN BitLocker.

Recommandations :

- utiliser un PIN mémorisable
- éviter les valeurs faibles comme 000000 ou 123456
- ne pas stocker le PIN dans un fichier public
- garder la clé de récupération BitLocker sous la main avant le premier redémarrage

### Étape 6 — Vérifier la configuration

Commande :

```cmd
manage-bde -protectors -get C:
```

Le résultat attendu doit contenir un protecteur de type :

```text
TPM et PIN
```

### Étape 7 — Tester au redémarrage

Commande :

```cmd
shutdown /r /t 0
```

Au redémarrage, Windows doit demander le PIN BitLocker avant le chargement complet du système.

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
[ ] Stratégie TPM + PIN configurée
[ ] Protecteur TPM + PIN ajouté
[ ] Redémarrage de test effectué
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
- une validation par une personne administratrice système ou sécurité

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
