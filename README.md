# misp-ioc-extractor

## Objectif projet 

ce projet consiste à apprendre la partie automatisation via python de l'extraction d'Indicateur de compromission (IoCs) depuis une instance MISP locale. Ce projet a été réalisé dans l'optique de m'initier a l'intéraction avec les API, l'utilisation de script et me renseigner sur de domaine du  Threat Intelligence


## Scénario (Threat Intel)
Ce projet se base sur l'analyse de l'événement **Microcin Malware**. Le script recherche un hash spécifique (MD5), identifie l'événement associé, et extrait automatiquement tous les attributs liés (IPs, Domaines, etc.) pour générer des rapports prêts à être ingérés par un SIEM ou un pare-feu.



##  Stack Technique
* **Langage :** Python 3
* **Librairies :** PyMISP, python-dotenv
* **Cible :** Plateforme MISP (Malware Information Sharing Platform)



##  Comment l'utiliser

1. Cloner le dépôt : `git clone https://github.com/Zaki-Nacer/misp-ioc-extractor.git`
2. Créer un environnement virtuel : `python -m venv venv`
3. Activer l'environnement : `.\venv\Scripts\activate`
4. Installer les dépendances : `pip install -r requirements.txt`
5. Créer un fichier `.env` avec les variables `MISP_URL` et `MISP_KEY`.
6. Lancer le script : `python extractor.py`

## 📊 Formats d'export
Le script génère deux types de fichiers de sortie :
* `iocs.json` : Pour une intégration automatisée via API.
* `iocs.csv` : Pour une analyse humaine rapide.




## Glossaire

**MISP** : Malware Information Sharing Platform est une platforme open-source de partage d'indicateurs de menace permettant aux organisations de collaborer sur les cyberattaques

**IoC** : Indicateur of Compromise est une preuve numérique d'une intrusion, ça peut être une ip, un hash ou un nom de domaine

**Hash** : une empreinte numérique d'un fichier, le Hash est unique a chaque fichier, une modification --> un hash différent

**Event** : Un dossier dans MISP qui regroupe toutes les informations et les IoCs lié à une menace spécifique

**API** : une interface logiciel qui agit comme un pont entre 2 logiciel ou service pour y échanger des données et fonctionnalités

**Variables d'Environnement (.env)** : méthode permettant de stocker des secrets (clé API, mdp) à l'extérieur du code source 

**Environnement Virtuel** : un espace isolé sur un système qui contient uniquement les librairie nécessaire à un projet spécifique afin d'éviter des conflit de version sur 2 projets différents


**DLL Hijacking** : une technique d'attaque consistant à placer un fichier malvaillant dans un dossier pour qu'il soit charger à la place d'un fichier légitime comme une bibliothèqyue par un logiciel

**Principe de Moindre Privilège** : Concepte consistant à donner le stric nécessaire de droit à un utilisateur ou à un script


## Fonctionnement général du script
* **1ère étape :** Se connecter à la VM et au MISP.
* **2ème étape :** Chercher un Hash choisi arbitrairement.
* **3ème étape :** Récupérer l'événement dont le Hash est issu.
* **4ème étape :** À l'aide de l'événement, on récupère le reste des IoCs (comme le hash de l'étape 3).
* **5ème étape :** Filtrer les IoCs importants (liés à un pare-feu, Antivirus ou EDR).
* **6ème étape :** Dresser un tableau des IoCs filtrés, sous format .json et .csv.



## Logique de l'algorithme 

### Recherche et Récupération de l'Événement

* **Connexion :** On initialise la connexion à l'API MISP avec nos clés.
* **Recherche :** On demande à MISP de chercher le Hash (notre point d'entrée).
* **Extraction de l'ID :** Si le Hash est trouvé, on extrait le numéro (`event_id`) du dossier parent.
* **Téléchargement :** On télécharge l'intégralité du dossier (l'Event) grâce à cet ID pour obtenir le tableau brut des IoCs.



### Filtrage des IoCs utiles 
* **Définition :** On définit les types d'attributs qu'on veut garder (ex: `ip-dst`, `domain`, `md5`).
* **Initialisation :** On crée un tableau `IoCs_filtres` qui est pour l'instant vide.*

*(Boucle  - FOR)*
* **Parcours :** On parcourt le tableau contenant tous les IoCs de l'événement ciblé.
* **Comparaison :** Pendant le parcours, on compare le `type` de chaque IoC avec notre liste définie à l'étape 1.
* **Injection :** Si le type correspond, on injecte l'IoC complet dans notre tableau filtré. Sinon, on l'ignore et on passe au suivant.
* **Fin :** On répète l'opération jusqu'à avoir lu le dernier élément du tableau.



### Génération des fichiers d'export (CSV et JSON)
* **Vérification :** On s'assure que le tableau filtré n'est pas vide.
* **Création du CSV :** On crée un fichier `iocs.csv`, on écrit les en-têtes (Type, Valeur, Catégorie), puis on insère chaque élément de notre tableau filtré ligne par ligne.
* **Création du JSON :** On crée un fichier `iocs.json` et on y injecte directement notre tableau formaté pour les machines (EDR/Pare-feu).
* **Clôture :** On ferme les connexions et on affiche un message de succès.