# SOC140 : Phishing Mail Detected – Suspicious Task Scheduler

[![Plateforme](https://img.shields.io/badge/Plateforme-LetsDefend-1abc9c?style=flat-square)](https://letsdefend.io/)
[![Catégorie](https://img.shields.io/badge/Catégorie-Phishing-e74c3c?style=flat-square)]()
[![Sévérité](https://img.shields.io/badge/Sévérité-Medium-e8a030?style=flat-square)]()
[![Verdict](https://img.shields.io/badge/Verdict-À_déterminer-c0c0c0?style=flat-square)]()

---

## Contexte de l'alerte

<img width="1505" height="433" alt="image" src="https://github.com/user-attachments/assets/21796482-e30d-414b-be9b-75b3eb00f7f3" />


## Contexte de l'alerte

| Champ                 | Valeur                                                 |
|-----------------------|--------------------------------------------------------|
| ID de l'événement     | 82                                                     |
| Date / heure          | 21 mars 2021, 12:26 PM                                 |
| Règle                 | SOC140 - Phishing Mail Detected - Suspicious Task Scheduler |
| Niveau                | Security Analyst                                       |
| Adresse SMTP          | 189.162.189.159                                       |
| Adresse source        | aaronluo@cmail.carleton.ca                            |
| Adresse de destination| mark@letsdefend.io                                    |
| Objet de l'e-mail     | COVID19 Vaccine                                       |
| Action de l'appareil| Blocked                                               |

> **Hypothèse de départ :** Un e‑mail au sujet sensible (*COVID19 Vaccine*) a été détecté comme potentiellement malveillant. L’objectif est de vérifier si ce mail contient des indicateurs de phishing (URL ou pièce jointe malveillante), de confirmer s’il a été livré ou bloqué, puis d’évaluer l’impact potentiel pour l’utilisateur cible.

### Étape 2 : Analyse du contenu de l'e-mail

En se rendant sur *Email Security*, j'obtiens le contenu du mail :
<img width="1556" height="598" alt="Capture d&#39;écran 2026-06-16 180355" src="https://github.com/user-attachments/assets/4b37584e-6ee5-46de-b760-c31dbd757c6e" />

L’e‑mail présente plusieurs signaux clairs de phishing :


- **Expéditeur :** `aaronluo@cmail.carleton.ca` (domaine externe, sans lien avec l’organisation)
- **Destinataire :** `mark@letsdefend.io`
- **Objet :** `COVID19 Vaccine` (thématique sensible / émotionnelle)
- **Corps du message :**
  > Hey, did you read breaking news about Covid-19. Open it now!

- Ton invitant à **l’urgence** (« Open it now! »), typique des campagnes de phishing visant à pousser l’utilisateur à ouvrir une pièce jointe ou cliquer sur un lien.
- Mention d’un **mot de passe** dans le corps du mail : `password: infected`, qui sert à déverrouiller la pièce jointe chiffrée.

> Ce message combine un sujet d’actualité anxiogène (COVID‑19), un sens de l’urgence et une pièce jointe protégée par mot de passe. C’est un schéma classique utilisé pour contourner certains filtres antispam/antivirus et inciter l’utilisateur à ouvrir un fichier potentiellement malveillant. Ce n'est déjà pas un bon signe.

### Étape 3 : Analyse de l’URL / pièce jointe dans des sandboxes tiers

Depuis le playbook, la prochaine étape consiste à analyser la pièce jointe dans des outils externes
tels que **VirusTotal**, **AnyRun**, **URLHaus**, **URLScan** ou **Hybrid Analysis**.

L’objectif est de répondre à une question simple :  
**la pièce jointe est‑elle malveillante ou non ?**

Dans le cadre de ce lab, j’ai décidé d'utiliser **VirusTotal**:

<img width="1780" height="245" alt="image" src="https://github.com/user-attachments/assets/710deeee-57ee-40b3-8d56-db05d000361c" />

Donc 
- **10/95** moteurs de sécurité marquent cette URL comme **malveillante**.
- Le fichier servi est un ZIP, typiquement utilisé pour
  transporter un document ou un exécutable piégé.

> **Conclusion de l’étape :** La réputation VirusTotal confirme que l’URL de téléchargement associée à la
> pièce jointe est malveillante. La plupart l'ont qualifié de **Trojan**.
> Heureusement, vu l'Action de l'appareil donné plus haut, **le mail ne semble pas avoir été distribué**.
