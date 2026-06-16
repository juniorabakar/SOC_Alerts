# SOC140 : Phishing Mail Detected – Suspicious Task Scheduler

[![Plateforme](https://img.shields.io/badge/Plateforme-LetsDefend-1abc9c?style=flat-square)](https://letsdefend.io/)
[![Catégorie](https://img.shields.io/badge/Catégorie-Phishing-e74c3c?style=flat-square)]()
[![Sévérité](https://img.shields.io/badge/Sévérité-Medium-e8a030?style=flat-square)]()
[![Verdict](https://img.shields.io/badge/Verdict-À_déterminer-c0c0c0?style=flat-square)]()

---

## Contexte de l'alerte

<img width="900" alt="Contexte alerte SOC140" src="(à remplacer par l’URL de ta capture)" />

| Champ                | Valeur                                                 |
|----------------------|--------------------------------------------------------|
| EventID              | 82                                                     |
| Event Time           | Mar 21, 2021, 12:26 PM                                 |
| Rule                 | SOC140 - Phishing Mail Detected - Suspicious Task Scheduler |
| Level                | Security Analyst                                       |
| SMTP Address         | `189.162.189.159`                                     |
| Source Address       | `aaronluo@cmail.carleton.ca`                          |
| Destination Address  | `mark@letsdefend.io`                                  |
| E-mail Subject       | `COVID19 Vaccine`                                     |
| Device Action        | `Blocked`                                             |

> **Hypothèse de départ :** Un e‑mail au sujet sensible (*COVID19 Vaccine*) a été détecté comme potentiellement malveillant. L’objectif est de vérifier si ce mail contient des indicateurs de phishing (URL ou pièce jointe malveillante), de confirmer s’il a été livré ou bloqué, puis d’évaluer l’impact potentiel pour l’utilisateur cible.
