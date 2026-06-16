# SOC141 : Phishing URL Detected

[![Plateforme](https://img.shields.io/badge/Plateforme-LetsDefend-1abc9c?style=flat-square)](https://letsdefend.io/)
[![Catégorie](https://img.shields.io/badge/Catégorie-Phishing-e74c3c?style=flat-square)]()
[![Sévérité](https://img.shields.io/badge/Sévérité-Medium-e8a030?style=flat-square)]()
[![Verdict](https://img.shields.io/badge/Verdict-Vrai_Positif-c0392b?style=flat-square)]()

---

## Contexte de l'alerte

<img width="1535" height="526" alt="Contexte alerte SOC141" src="https://github.com/user-attachments/assets/812134d0-6b6a-4eb1-8160-eb721633ff8f" />

| Champ | Valeur |
|-------|--------|
| Règle déclenchée | Phishing URL Detected |
| Action | `Allowed` |
| Vecteur | Clic sur URL dans le navigateur |
| Contexte | Tentative d'accès à une URL suspecte détectée par le proxy |

> **Hypothèse de départ :** un utilisateur a cliqué sur une URL potentiellement malveillante. L’objectif est de déterminer si l’URL est réellement malveillante, si l’accès a abouti, et d’identifier l’utilisateur et la machine à l’origine de la requête, afin d’évaluer le risque et d’appliquer les mesures de confinement appropriées.

---

## Méthodologie de triage

### Étape 1 : Collecte et qualification des indicateurs

La première étape consiste à extraire les informations essentielles fournies par l’alerte :

- **Adresse source :** `172.16.17.49`
- **Adresse de destination :** `91.189.114.8`
- **User-Agent :** `Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36`
- **Type d’événement :** accès HTTP vers une URL suspecte

Ces éléments servent de pivot pour rechercher des événements corrélés dans le **Log Management** (même IP source, même IP destination, même User-Agent, même fenêtre temporelle).

<img width="1461" height="116" alt="Log Management" src="https://github.com/user-attachments/assets/bb2c8a95-e9d9-4f59-b4d2-c56883ca136d" />

En affinant la recherche, on identifie la requête HTTP exacte et donc l’URL incriminée.

<img width="749" height="247" alt="URL suspecte" src="https://github.com/user-attachments/assets/aadd5352-a23e-42bb-a89e-2c94096b703d" />

---

### Étape 2 : Analyse de l’URL via VirusTotal

Une fois l’URL identifiée, elle est soumise à **VirusTotal**, qui agrège les verdicts de nombreux moteurs antivirus et services de réputation pour les fichiers, URLs, domaines et IPs.

<img width="1625" height="224" alt="VirusTotal" src="https://github.com/user-attachments/assets/c2b76d51-bb60-4bc4-885a-e836dd90a2d0" />

**Résultat de l’analyse :**

- **12/97** moteurs classent l’URL comme malveillante/suspecte.
- **85/97** ne la signalent pas.
- VirusTotal obtient une réponse **403 Forbidden** lors de la récupération du contenu (le serveur refuse l’accès).
- L’URL pointe vers le domaine `mogagrocol.ru` avec un chemin de type WordPress (`/wp-content/plugins/akismet/...`) et un paramètre `email=...@letsdefend.io`.

> **Interprétation :**  
> - Un score non nul (ici 12/97) est suffisant pour considérer l’URL comme **suspecte** dans un contexte SOC, même en l’absence de consensus complet.  
> - Le code **403** ne signifie pas que l’URL est sûre ; il indique simplement que le contenu n’a pas pu être servi à VirusTotal, ce qui limite l’analyse automatisée.  
> - Le combo domaine `.ru` + chemin WordPress + paramètre email est typique soit d’un site compromis, soit d’une campagne de phishing utilisant un tracking ciblé.

---

### Étape 3 : Vérification des accès dans les journaux internes

Objectif : **confirmer si l’URL a effectivement été atteinte** par un poste interne.

<img width="1462" height="227" alt="Vérification journaux" src="https://github.com/user-attachments/assets/09ffd0fe-d542-4df6-a23f-191c21bc2730" />

Les journaux montrent que la requête HTTP a été **autorisée** (`Allowed`) et que l’URL a bien été contactée.

| Champ | Valeur |
|-------|--------|
| Date d'accès | 22 mars 2021 à 21h23 |
| Adresse source | `172.16.17.49` |
| Adresse de destination | `91.189.114.8` |
| Action proxy | `Allowed` |
| Accès à l'URL | Oui |
| User-Agent | `Mozilla/5.0 (Windows NT 6.1; Win64; x64)...` |

> **Conclusion de l’étape :** il ne s’agit pas d’une simple tentative bloquée, mais bien d’un accès effectif à une URL classée suspecte par plusieurs moteurs.

---

### Étape 4 : Corrélation EDR et identification de l’utilisateur

L’étape suivante consiste à corréler l’adresse IP source avec les informations de l’EDR pour identifier :

- la machine concernée,
- l’utilisateur connecté au moment des faits.

<img width="1515" height="725" alt="EDR" src="https://github.com/user-attachments/assets/cd14d909-ac47-467e-a1b3-cc44716cfba7" />

Résultat :

- **Machine :** `EmilyComp`
- **Utilisateur :** `Emily`

Cette corrélation est essentielle pour toute action de réponse à incident (confinement, communication utilisateur, investigations complémentaires).

---

### Étape 5 : Confinement de la machine

L’alerte étant classée **Vrai Positif** et l’accès à l’URL confirmé, la machine `EmilyComp` est placée en **confinement** depuis la console EDR.

<img width="1017" height="595" alt="Confinement" src="https://github.com/user-attachments/assets/b9ef46a9-c7ec-449f-a887-0f77186319cb" />

Ce confinement permet :

- de limiter toute communication sortante potentiellement malveillante,
- d’éviter une éventuelle latéralisation si un payload a été téléchargé,
- de donner le temps aux équipes de réaliser des analyses complémentaires (EDR, AV, forensique).

---

### Étape 6 : Verdict et clôture

| Critère | Résultat |
|---------|----------|
| Réputation de l’URL | ✅ 12/97 moteurs VT la marquent malveillante/suspecte |
| Accès effectif | ✅ Confirmé dans les logs (action `Allowed`) |
| Machine et utilisateur identifiés | ✅ `EmilyComp` / `Emily` |
| Mesures de confinement | ✅ Machine isolée via EDR |
| Impact utilisateur | À évaluer (éventuelle exposition aux données de phishing) |

**Verdict : Vrai Positif (True Positive).**  
L’URL est considérée comme suspecte/malveillante et un utilisateur y a effectivement accédé. L’incident justifie un confinement et une investigation complémentaire.

---

## Leçons apprises

### 1. Indicateurs de compromission (IoCs)

| Type | Valeur |
|------|--------|
| IP source | `172.16.17.49` |
| IP destination | `91.189.114.8` |
| Domaine | `mogagrocol.ru` |
| Machine | `EmilyComp` |
| Utilisateur | `Emily` |
| User-Agent | `Mozilla/5.0 (Windows NT 6.1; Win64; x64)...` |

### 2. Ce que ça signifie en contexte GRC/SOC

- **GRC** : Ce cas illustre la nécessité de contrôles efficaces sur la navigation web (filtrage URL, proxy, DNS) en cohérence avec les exigences de sécurité réseau et de protection contre les logiciels malveillants.
- **SOC** : Un score VirusTotal partiellement consensuel ne doit pas être négligé. Dans une logique SOC, il est préférable de sur-classer une URL douteuse comme suspecte plutôt que d’ignorer des signaux faibles. La corrélation systématique entre logs réseau, EDR et identité utilisateur est indispensable pour justifier le verdict et tracer le raisonnement.

### 3. Recommandations de remédiation

> *Ces recommandations sont formulées dans le cadre d’un lab fictif et ne sont pas implémentées en production.*

- [x] Bloquer le domaine `mogagrocol.ru` et l’IP `91.189.114.8` au niveau du proxy/firewall.
- [x] Maintenir **`EmilyComp`** en confinement le temps d’une analyse complémentaire (EDR, AV, logs).
- [x] Vérifier via le SIEM si d’autres machines ont tenté d’accéder à la même URL ou au même domaine sur la période récente.
- [x] Mettre à jour les règles de filtrage web (proxy/DNS) pour renforcer le blocage des domaines à faible réputation.
- [x] Sensibiliser l’utilisateur `Emily` (et plus largement les utilisateurs) aux risques du phishing et aux bonnes pratiques (vérification des URLs, signalement des mails suspects).
- [x] Documenter l’IoC (domaine, IP, URL complète) dans la base de connaissances interne et dans les playbooks SOC.

---

[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1566_Phishing-E74C3C?style=flat-square)](https://attack.mitre.org/techniques/T1566/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1204_User_Execution-E74C3C?style=flat-square)](https://attack.mitre.org/techniques/T1204/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1071_App_Layer_Protocol-E74C3C?style=flat-square)](https://attack.mitre.org/techniques/T1071/)
[![VirusTotal](https://img.shields.io/badge/VirusTotal-Documentation-394EFF?style=flat-square)](https://docs.virustotal.com/)
