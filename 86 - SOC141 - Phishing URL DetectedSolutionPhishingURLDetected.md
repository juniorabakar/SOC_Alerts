Le Contexte est donné dans l'image suivante.
<img width="1535" height="526" alt="image" src="https://github.com/user-attachments/assets/812134d0-6b6a-4eb1-8160-eb721633ff8f" />

Prenons alors en charge cette alerte!

Pour commencer, on devrait vérifier les détails suivants dans l'alerte:
- Adresse source (Ici, 172.16.17.49)
- Adresse de destination (Ici, 91.189.114.8)
- Le User-Agent (Ici, Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36).

Avec ces 3 informations essentielles, rendons-nous maintenant dans le log management afin de rechercher plus de détails. Nous y trouvons deux entrées correspondant à
la date de l'alerte:

<img width="1461" height="116" alt="image" src="https://github.com/user-attachments/assets/bb2c8a95-e9d9-4f59-b4d2-c56883ca136d" />

En y regardant de plus près, on voit qu'une URL a été émise depuis l'adresse IP suspecte.

<img width="749" height="247" alt="image" src="https://github.com/user-attachments/assets/aadd5352-a23e-42bb-a89e-2c94096b703d" />

Il faut donc analyser cette adresse URL à l'aide d'outils tiers afin de déterminer si l'URL est malveillante ou ne l'est pas. Pour ce faire, nous disposons de plusieurs
outils: 
AnyRun
VirusTotal
URLHouse
URLScan
HybridAnalysis

Ayant un penchant pour VirusTotal, j'utiliserai cet outil. Voici les résultats de mon analyse au moment de mon scan.

<img width="1625" height="224" alt="image" src="https://github.com/user-attachments/assets/c2b76d51-bb60-4bc4-885a-e836dd90a2d0" />

On voit alors que 12 moteurs antivirus/solutions sur 97 considèrent l’URL comme malveillante/suspecte, mais la majorité (85/97) ne la flag pas (<i>ce qui peut arriver pour du phishing “nouveau”, du low-confidence, ou des détections divergentes</i>).
De plus, quand VirusTotal a tenté de récupérer l’URL, le serveur a répondu Forbidden (accès refusé), donc le contenu n’a peut‑être pas pu être analysé correctement.
Par ailleurs, L’URL pointée est sur le domaine mogagrocol.ru et ressemble à un chemin WordPress (/wp-content/plugins/akismet/...) avec un paramètre email=...@letsdefend.io, ce qui peut être typique d’une URL utilisée dans une campagne (tracking/phishing) ou d’un site compromis.​

Quoiqu'il en soit, avec 12/97 détections sur VirusTotal, c’est raisonnable de considérer l’URL comme malveillante dans un contexte SOC (à minima “suspicious/malicious”) plutôt que “non-malicious”.

On doit se demander maintenant si quelqu'un a accédé à l'adresse IP/URL/domaine en vérifiant dans la gestion des journaux s'il existe un appareil capable d'accéder à ces adresses à partir des appareils du réseau.

<img width="1462" height="227" alt="image" src="https://github.com/user-attachments/assets/09ffd0fe-d542-4df6-a23f-191c21bc2730" />

Et si c'est le cas, trouver les réponses aux questions suivantes:
- Quand y a-t-il eu accès ? <b> Le 22 Mars 2021 à 09:23 PM </b>
- Quelle est l'adresse source ? <b> 172.16.17.49 </b>
- Quelle est l'adresse de destination ? <b> 91.189.114.8 </b>
- Quel utilisateur a tenté d'y accéder ?

<img width="1515" height="725" alt="image" src="https://github.com/user-attachments/assets/cd14d909-ac47-467e-a1b3-cc44716cfba7" />

En allant dans la solution EDR, <b> on voit qu'il s'agit de l'utilisateur Emily avec le nom d'hôte EmilyComp </b>

- Quel est l'agent utilisateur ? <b> Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36 </b>
- La requête est-elle bloquée ? <b> Non, on voit bien que l'action dans l'alerte est Allowed </b>
- Y a-t-il un accès à l'URL ? <b> Oui </b>

Vu que l'alerte est potentiellement un vrai positif, on doit se rendre sur la page « EDR » et confiner la machine de l'utilisateur !
<img width="1017" height="595" alt="image" src="https://github.com/user-attachments/assets/b9ef46a9-c7ec-449f-a887-0f77186319cb" />

Enfin, rajoutons les artefacts importants ainsi que les notes pour aider les analystes de niveau 2.
