
# 🔐 Rapport Complet – Analyse Statique APK
## OWASP UnCrackable Level 1

---

# 📌 Introduction

Ce dépôt présente la réalisation complète du laboratoire d’analyse statique d’une application Android :

**OWASP MSTG – UnCrackable Level 1**

Objectifs du lab :

- Préparer un environnement de travail structuré
- Vérifier l’intégrité de l’APK
- Analyser l’APK avec JADX GUI
- Identifier des informations sensibles
- Convertir DEX → JAR avec dex2jar
- Comparer les outils de décompilation
- Produire un mini‑rapport professionnel

---

# 🧩 Task 1 — Préparer le Workspace et Vérifier l’APK

## 1️⃣ Création du dossier de travail

```powershell
mkdir C:\APK-Analysis
cd C:\APK-Analysis
```

Objectif : Travailler dans un environnement propre et isolé.

---

## 2️⃣ Vérification que l’APK est une archive ZIP

```powershell
Get-Content -Path .\UnCrackable-Level1.apk -TotalCount 4 | Format-Hex
```

Résultat attendu :

Les 4 premiers octets doivent commencer par :

50 4B → Signature ZIP (PK)

📸 Capture :
![HEX APK](docs/images/1.png)

Conclusion : L’APK est bien une archive ZIP valide.

---

## 3️⃣ Liste du contenu interne de l’APK

```powershell
Add-Type -Assembly System.IO.Compression.FileSystem
$apk = Join-Path (Get-Location).Path "UnCrackable-Level1.apk"
[System.IO.Compression.ZipFile]::OpenRead($apk).Entries | Select-Object -ExpandProperty FullName -First 20
```

Éléments observés :
- AndroidManifest.xml
- classes.dex
- resources.arsc
- META-INF/
- res/

📸 Capture :
![Liste APK](docs/images/2.png)

Conclusion : Structure standard d’une application Android identifiée.

---

## 4️⃣ Calcul du hash SHA‑256

```powershell
Get-FileHash -Algorithm SHA256 .\UnCrackable-Level1.apk
```

Hash obtenu :

1DA8BF57D266109F9A07C01BF7111A1975CE01F190B9D914BCD3AE3DBEF96F21

📸 Capture :
![SHA256](docs/images/3.png)

Importance :
Le hash garantit l’intégrité et la traçabilité du fichier analysé.

---

# 📦 Task 2 — Obtention de l’APK

Source utilisée :
OWASP MSTG Crackmes

Lien :
https://mas.owasp.org/crackmes/Android/

L’APK a été téléchargé puis copié dans le dossier de travail.

📸 Capture ouverture dans JADX :
![Open APK](docs/images/4.png)

---

# 🔎 Task 3 — Analyse avec JADX GUI

## 1️⃣ Ouverture de l’APK

L’APK est ouvert dans JADX GUI pour explorer :
- Le manifeste
- Les ressources
- Le code source décompilé

📸 Vue générale :
![Structure JADX](docs/images/5.png)

---

## 2️⃣ Analyse du AndroidManifest.xml

Informations identifiées :

- Package : owasp.mstg.uncrackable1
- versionName : 1.0
- minSdkVersion : 19
- targetSdkVersion : 28
- Activity principale : sg.vantagepoint.uncrackable1.MainActivity
- Intent-filter : MAIN + LAUNCHER

📸 Capture manifeste :
![Manifest](docs/images/6.png)

Analyse sécurité :
- Aucune permission dangereuse
- Pas de android:debuggable="true"
- Pas de cleartextTraffic activé

---

## 3️⃣ Analyse des ressources (strings.xml)

Chaînes importantes trouvées :

- "Enter the Secret String"
- "Verify"
- "This is the correct secret."

📸 Capture :
![Strings](docs/images/7.png)

Observation :
Présence d’un message de succès codé en clair.

---

# 🔍 Task 4 — Recherche de chaînes sensibles

Recherche globale effectuée avec Ctrl+Shift+F dans JADX.

## 🔎 Recherche http://

📸
![HTTP](docs/images/8.png)

Résultat : Aucun endpoint externe détecté.

---

## 🔎 Recherche .com

📸
![COM](docs/images/9.png)

Résultat : Aucun serveur distant identifié.

---

## 🔎 Recherche dev

📸
![DEV](docs/images/10.png)

Observation :
Référence à des chemins système Android.

---

## 🔎 Recherche secret

📸
![SECRET](docs/images/11.png)

Observation importante :
Utilisation de :
AES/ECB/PKCS7Padding
SecretKeySpec

Présence d’un mécanisme de vérification du secret.

---

## 🔎 Recherche debug

📸
![DEBUG](docs/images/12.png)

Observation :
Message "App is debuggable!" présent dans le code.

---

# 📊 Observations de Sécurité

| Constat | Sévérité | Description |
|----------|----------|-------------|
| Détection Root | Moyen | Empêche l’exécution sur appareil rooté |
| Détection Debug | Moyen | Empêche le debugging |
| Secret validé côté client | Élevé | Logique sensible côté application |
| AES ECB utilisé | Moyen | Mode ECB faible en cryptographie |
| Message succès en clair | Faible | Divulgation d’information |

---

# 🔄 Task 5 — Conversion DEX → JAR

## Extraction des fichiers DEX

📸
![DEX Extract](docs/images/13.png)

---

## Conversion avec dex2jar

```powershell
d2j-dex2jar.bat classes.dex -o app.jar
```

📸
![DEX2JAR](docs/images/14.png)

Résultat :
Fichier app.jar généré avec succès.

---

# 🔎 Task 6 — Comparaison JADX vs JD-GUI

Ouverture du JAR dans JD-GUI.

📸
![JDGUI](docs/images/15.png)

---

## Comparaison détaillée

| Critère | JADX | JD-GUI |
|----------|--------|--------|
| Ressources Android | Oui | Non |
| Manifest | Oui | Non |
| Lisibilité Android | Excellente | Moyenne |
| Vue Java pure | Bonne | Excellente |
| Gestion obfuscation | Améliorée | Limitée |

Conclusion :
JADX est plus adapté à l’analyse Android.
JD-GUI est utile pour l’analyse Java classique.

---

# 📄 Task 7 — Mini Rapport Professionnel

## Informations générales

- Application : OWASP UnCrackable Level 1
- Version : 1.0
- Analyste : Mahmoud Laasri
- Outils : JADX, dex2jar, JD-GUI
- Hash SHA‑256 : 1DA8BF57D266109F9A07C01BF7111A1975CE01F190B9D914BCD3AE3DBEF96F21

---

## Résumé exécutif

L’analyse statique révèle plusieurs mécanismes de protection implémentés :

- Détection Root
- Détection Debug
- Vérification d’un secret
- Utilisation d’un chiffrement AES

Il s’agit d’une application volontairement protégée (challenge sécurité).

Niveau de risque global : Moyen

---

## Constats détaillés

### Constat 1 — Détection Root
Sévérité : Moyenne  
Impact : Empêche l’analyse dynamique  
Remédiation : Déporter les contrôles critiques côté serveur

### Constat 2 — Détection Debug
Sévérité : Moyenne  
Impact : Bloque le reverse engineering  
Remédiation : Renforcer la protection par obfuscation avancée

### Constat 3 — Logique sensible côté client
Sévérité : Élevée  
Impact : Permet le reverse engineering du mécanisme  
Remédiation : Implémenter validation serveur

---

# ✅ Validation finale

✔ Workspace structuré  
✔ Intégrité vérifiée  
✔ Analyse manifeste effectuée  
✔ Recherche chaînes sensibles réalisée  
✔ DEX extrait  
✔ JAR généré  
✔ Comparaison outils documentée  
✔ Rapport rédigé  

---

# 👨‍💻 Auteur

Mahmoud Laasri  
Analyse Statique Mobile – Sécurité Android
