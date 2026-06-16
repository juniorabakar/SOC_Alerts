# SOC140 : Phishing Mail Detected – Suspicious Task Scheduler

[![Plateforme](https://img.shields.io/badge/Plateforme-LetsDefend-1abc9c?style=flat-square)](https://letsdefend.io/)
[![Catégorie](https://img.shields.io/badge/Catégorie-Phishing-e74c3c?style=flat-square)]()
[![Sévérité](https://img.shields.io/badge/Sévérité-Medium-e8a030?style=flat-square)]()
[![Verdict](https://img.shields.io/badge/Verdict-Vrai_Positif-c0392b?style=flat-square)]()

---

## Contexte de l'alerte

<img width="1505" height="433" alt="Contexte alerte SOC140" src="https://github.com/user-attachments/assets/21796482-e30d-414b-be9b-75b3eb00f7f3" />

| Champ                  | Valeur                                                 |
|------------------------|--------------------------------------------------------|
| ID de l'événement      | 82                                                     |
| Date / heure           | 21 mars 2021, 12:26 PM                                 |
| Règle                  | SOC140 - Phishing Mail Detected - Suspicious Task Scheduler |
| Niveau                 | Security Analyst                                       |
| Adresse SMTP           | 189.162.189.159                                        |
| Adresse source         | aaronluo@cmail.carleton.ca                             |
| Adresse de destination | mark@letsdefend.io                                     |
| Objet de l'e-mail      | COVID19 Vaccine                                        |
| Action de l'appareil   | Blocked                                                |

> **Hypothèse de départ :** un e‑mail au sujet sensible (*COVID19 Vaccine*) a été détecté comme potentiellement malveillant. L’objectif est de vérifier si ce mail contient des indicateurs de phishing (URL ou pièce jointe malveillante), de confirmer s’il a été livré ou bloqué, puis d’évaluer l’impact potentiel pour l’utilisateur cible.

---

## Étape 2 : Analyse du contenu de l’e-mail

En se rendant sur **Email Security**, on récupère le contenu complet du message :

<img width="1556" height="598" alt="Contenu de l'e-mail" src="https://github.com/user-attachments/assets/4b37584e-6ee5-46de-b760-c31dbd757c6e" />

L’e‑mail présente plusieurs signaux clairs de phishing :

- **Expéditeur :** `aaronluo@cmail.carleton.ca` (domaine externe, sans lien avec l’organisation).
- **Destinataire :** `mark@letsdefend.io`.
- **Objet :** `COVID19 Vaccine` (thématique sensible / émotionnelle).
- **Corps du message :**  
  > Hey, did you read breaking news about Covid-19. Open it now!

Éléments supplémentaires :

- Ton invitant à **l’urgence** (« Open it now! »), typique des campagnes de phishing visant à pousser l’utilisateur à ouvrir une pièce jointe ou cliquer sur un lien.
- Mention d’un **mot de passe** dans le corps du mail : `password: infected`, qui sert à déverrouiller la pièce jointe chiffrée.

> Ce message combine un sujet d’actualité anxiogène (COVID‑19), un sens de l’urgence et une pièce jointe protégée par mot de passe. C’est un schéma classique utilisé pour contourner certains filtres antispam/antivirus et inciter l’utilisateur à ouvrir un fichier potentiellement malveillant.

---

## Étape 3 : Analyse de l’URL / pièce jointe dans des sandboxes tiers

Le playbook LetsDefend demande ensuite d’analyser la pièce jointe ou son URL de téléchargement dans des outils externes tels que **VirusTotal**, **AnyRun**, **URLHaus**, **URLScan** ou **Hybrid Analysis** [web:79].

L’objectif est de répondre à une question simple :  
**la pièce jointe est‑elle malveillante ou non ?**

Dans le cadre de ce lab, j’ai utilisé **VirusTotal** pour analyser l’URL de téléchargement de la pièce jointe, hébergée sur un bucket **Amazon S3** :

<img width="1780" height="245" alt="Analyse VirusTotal de l'URL" src="https://github.com/user-attachments/assets/710deeee-57ee-40b3-8d56-db05d000361c" />

Les résultats montrent que :

- **10/95** moteurs de sécurité marquent cette URL comme **malveillante**.
- L’URL pointe vers un fichier ZIP, format fréquemment utilisé pour transporter un document ou un exécutable piégé.
- La plupart des moteurs l’identifient comme un **Trojan**, ce qui confirme la nature malveillante de la charge.

> **Conclusion de l’étape :** la réputation VirusTotal confirme que l’URL de téléchargement associée à la pièce jointe est malveillante. Couplée au contenu du mail, cette URL s’inscrit clairement dans une campagne de phishing distribuant un fichier malveillant.

---

## Étape 4 : Note de l’analyste

L’alerte SOC140 signale un e‑mail suspect ayant pour objet « COVID19 Vaccine », envoyé depuis `aaronluo@cmail.carleton.ca` vers `mark@letsdefend.io`.  
L’analyse du contenu montre plusieurs indicateurs de phishing : thème COVID‑19, ton urgent (« Open it now! ») et présence d’une pièce jointe protégée par mot de passe (`password: infected`).  

La pièce jointe est accessible via une URL de téléchargement hébergée sur Amazon S3. Cette URL, soumise à VirusTotal, est détectée comme malveillante par **10/95** moteurs, majoritairement classée comme **Trojan**, ce qui confirme la nature malveillante du fichier distribué [web:67][web:88].  

D’après le champ **Device Action = Blocked**, l’e‑mail a été intercepté par la passerelle de messagerie avant livraison à l’utilisateur : aucun endpoint interne n’a donc exécuté la pièce jointe.  

Les principaux **IoCs** sont :
- l’adresse IP SMTP `189.162.189.159`,
- le couple expéditeur / destinataire (`aaronluo@cmail.carleton.ca` → `mark@letsdefend.io`),
- l’URL de téléchargement Amazon S3 du ZIP,
- le nom et le hash de la pièce jointe ZIP malveillante.

**Verdict : Vrai Positif (True Positive).**  
E‑mail de phishing avec pièce jointe malveillante correctement bloqué par les contrôles de sécurité mail.  
Aucune compromission constatée côté poste, mais les IoCs doivent être ajoutés aux listes de blocage et aux règles de détection, et l’utilisateur cible doit être sensibilisé aux campagnes de phishing thématisées COVID‑19.

---

## Leçons apprises

- Un e‑mail peut être un **vrai positif** même si la menace est bloquée en amont (ici au niveau de la passerelle Exchange) : le contrôle a fonctionné, mais la campagne est réelle [web:63][web:67].
- Les pièces jointes protégées par mot de passe, combinées à un sujet sensible et à un ton urgent, doivent systématiquement être considérées comme suspectes.
- L’analyse de l’URL de téléchargement dans VirusTotal ou une sandbox fournit rapidement un verdict sans exposer l’environnement interne.
- La vérification du champ **Device Action** est essentielle pour distinguer un mail **livré** d’un mail **bloqué**, et donc ajuster la profondeur de l’investigation côté endpoint.

---

[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1566_Phishing-E74C3C?style=flat-square)](https://attack.mitre.org/techniques/T1566/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1204.002_User_Execution_(Malicious_File)-E74C3C?style=flat-square)](https://attack.mitre.org/techniques/T1204/002/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1053.005_Scheduled_Task_Job-E74C3C?style=flat-square)](https://attack.mitre.org/techniques/T1053/005/)
