# SOC141 : Phishing URL Detected

[![Plateforme](https://img.shields.io/badge/Plateforme-LetsDefend-1abc9c?style=flat-square)](https://letsdefend.io/)
[![Catégorie](https://img.shields.io/badge/Catégorie-Phishing-e74c3c?style=flat-square)]()
[![Sévérité](https://img.shields.io/badge/Sévérité-Medium-e8a030?style=flat-square)]()
[![Verdict](https://img.shields.io/badge/Verdict-Vrai_Positif-c0392b?style=flat-square)]()

---

## Contexte de l'alerte

Le contexte est donné dans l'image suivante.

<img width="1535" height="526" alt="Contexte alerte" src="https://github.com/user-attachments/assets/812134d0-6b6a-4eb1-8160-eb721633ff8f" />

Prenons en charge cette alerte.

---

## Étape 1 : Collecte des informations initiales

La première chose à faire est d'extraire les informations clés directement depuis l'alerte :

- **Adresse source :** `172.16.17.49`
- **Adresse de destination :** `91.189.114.8`
- **User-Agent :** `Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36`

---

## Étape 2 : Recherche dans le Log Management

Avec ces trois informations en main, direction le **Log Management** pour chercher des entrées correspondant à la date de l'alerte. Deux entrées correspondent :

<img width="1461" height="116" alt="Log Management" src="https://github.com/user-attachments/assets/bb2c8a95-e9d9-4f59-b4d2-c56883ca136d" />

En regardant de plus près, une URL a été émise depuis l'adresse IP suspecte :

<img width="749" height="247" alt="URL suspecte" src="https://github.com/user-attachments/assets/aadd5352-a23e-42bb-a89e-2c94096b703d" />

---

## Étape 3 : Analyse de l'URL avec VirusTotal

Il faut maintenant analyser cette URL via des outils tiers pour déterminer si elle est malveillante. Plusieurs options sont disponibles :

- [AnyRun](https://any.run/)
- [VirusTotal](https://www.virustotal.com/)
- [URLHaus](https://urlhaus.abuse.ch/)
- [URLScan](https://urlscan.io/)
- [Hybrid Analysis](https://www.hybrid-analysis.com/)

J'opte pour **VirusTotal**. Voici les résultats au moment de mon analyse :

<img width="1625" height="224" alt="VirusTotal" src="https://github.com/user-attachments/assets/c2b76d51-bb60-4bc4-885a-e836dd90a2d0" />

**Analyse des résultats :**

- **12/97** moteurs antivirus considèrent l'URL comme malveillante ou suspecte
- **85/97** ne la flaggent pas : ce qui peut s'expliquer par du phishing récent, un faible niveau de confiance, ou des détections divergentes entre moteurs
- VirusTotal a reçu une réponse **Forbidden (403)** en tentant de récupérer l'URL : le contenu n'a peut-être pas pu être analysé correctement
- L'URL pointe vers le domaine **`mogagrocol.ru`** avec un chemin de type WordPress (`/wp-content/plugins/akismet/...`) et un paramètre `email=...@letsdefend.io` : typique d'une campagne de tracking/phishing ou d'un site compromis

> **Verdict :** Avec 12/97 détections, il est raisonnable de classifier cette URL comme **suspecte/malveillante** dans un contexte SOC plutôt que de la considérer comme bénigne.

---

## Étape 4 : Vérification des accès dans les journaux

La question clé est maintenant : **est-ce qu'un appareil du réseau a réellement accédé à cette URL ?**

<img width="1462" height="227" alt="Vérification journaux" src="https://github.com/user-attachments/assets/09ffd0fe-d542-4df6-a23f-191c21bc2730" />

La réponse est **oui**. Voici les détails :

| Champ | Valeur |
|-------|--------|
| **Date d'accès** | 22 Mars 2021 à 21h23 |
| **Adresse source** | `172.16.17.49` |
| **Adresse de destination** | `91.189.114.8` |
| **Requête bloquée ?** | ❌ Non : action `Allowed` |
| **Accès à l'URL ?** | ✅ Oui |
| **User-Agent** | `Mozilla/5.0 (Windows NT 6.1; Win64; x64)...` |

---

## Étape 5 : Identification de l'utilisateur via l'EDR

<img width="1515" height="725" alt="EDR" src="https://github.com/user-attachments/assets/cd14d909-ac47-467e-a1b3-cc44716cfba7" />

En consultant la solution EDR, on identifie que l'accès provient de l'utilisateur **Emily** sur la machine **EmilyComp**.

---

## Étape 6 : Confinement de la machine

L'alerte étant un **vrai positif**, on passe immédiatement au confinement de la machine d'Emily depuis la page EDR :

<img width="1017" height="595" alt="Confinement" src="https://github.com/user-attachments/assets/b9ef46a9-c7ec-449f-a887-0f77186319cb" />

---

## Étape 7 : Clôture de l'alerte

Pour finir, on renseigne les **artefacts importants** et on ajoute des **notes à destination des analystes de niveau 2** pour faciliter l'escalade si nécessaire.

---

## Leçons apprises

**1. Indicateurs de compromission (IoCs)**

| Type | Valeur |
|------|--------|
| IP source | `172.16.17.49` |
| IP destination | `91.189.114.8` |
| Domaine | `mogagrocol.ru` |
| Machine compromise | `EmilyComp` |
| Utilisateur | `Emily` |

**2. Ce que cette alerte m'a appris**

- Un score VirusTotal de 12/97 est suffisant pour considérer une URL comme suspecte dans un contexte SOC : ne pas attendre une majorité de détections
- Une réponse 403 de VirusTotal ne signifie pas que l'URL est saine : elle peut simplement indiquer que le contenu n'a pas pu être analysé
- Un domaine `.ru` avec un chemin WordPress et un paramètre `email=` est un signal fort de phishing ou de site compromis
- Le fait que la requête soit `Allowed` (non bloquée) rend le confinement immédiat de la machine indispensable

**3. Recommandations**

- [x] Bloquer l'IP `91.189.114.8` et le domaine `mogagrocol.ru` au niveau du proxy/firewall
- [x] Confiner la machine `EmilyComp` immédiatement
- [x] Analyser les autres machines du réseau pour détecter des connexions similaires
- [x] Sensibiliser l'utilisateur Emily aux risques du phishing



