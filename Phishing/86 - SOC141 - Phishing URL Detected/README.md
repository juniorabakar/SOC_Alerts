# Phishing — Alertes SOC

[![Plateforme](https://img.shields.io/badge/Plateforme-LetsDefend-1abc9c?style=flat-square)](https://letsdefend.io/)
[![Catégorie](https://img.shields.io/badge/Catégorie-Phishing-e74c3c?style=flat-square)]()

---

## À propos

Ce dossier regroupe les write-ups des alertes de catégorie **Phishing** traitées dans le cadre du SOC Analyst Learning Path sur LetsDefend.

Chaque write-up documente l'intégralité de la démarche d'investigation :
- Collecte des informations initiales depuis l'alerte
- Recherche dans les logs et l'EDR
- Analyse des URLs et domaines suspects via des outils tiers
- Verdict et confinement si nécessaire
- Leçons apprises et recommandations de remédiation

---

## Alertes traitées

| ID | Titre | Verdict | Outils utilisés |
|----|-------|---------|-----------------|
| [SOC141](./SOC141_Phishing_URL_Detected.md) | Phishing URL Detected | `Vrai Positif` | VirusTotal · Log Management · EDR |

---

## Outils d'analyse utilisés

| Outil | Usage |
|-------|-------|
| [VirusTotal](https://www.virustotal.com/) | Analyse d'URLs, domaines et IPs |
| [URLScan](https://urlscan.io/) | Scan et capture d'URLs suspectes |
| [URLHaus](https://urlhaus.abuse.ch/) | Base de données d'URLs malveillantes |
| [Any.run](https://any.run/) | Analyse comportementale en sandbox |
| [Hybrid Analysis](https://www.hybrid-analysis.com/) | Analyse statique et dynamique |

---

> *Ces write-ups sont réalisés dans un environnement simulé fourni par LetsDefend.*
