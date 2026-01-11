# TP 32 : Pipeline CI/CD Microservices avec Jenkins, GitHub, SonarQube, Docker Compose et Ngrok

Ce projet présente la mise en place complète d'un pipeline CI/CD pour une architecture de microservices utilisant Jenkins, GitHub, SonarQube, Docker Compose et Ngrok.



##  Objectifs pédagogiques

- Mettre en place Jenkins et configurer les outils (JDK, Maven, SonarScanner)
- Déployer SonarQube via Docker Compose et créer des projets + tokens par microservice
- Exposer Jenkins avec Ngrok et brancher GitHub via webhooks
- Créer un job Pipeline Jenkins et écrire un script de pipeline multi-stages
- Lancer/valider l'exécution (Jenkins, SonarQube, Docker) et vérifier le déclenchement par push

##  Prérequis

### Prérequis techniques (outils)

- **JDK 17** (ou version compatible) + variable `JAVA_HOME` configurée
- **Maven** (installé localement ou géré par Jenkins)
- **Git** (ligne de commande)
- **Docker** + **Docker Compose**
- **Jenkins** (installation locale)
- **SonarQube** (déployé via Docker Compose) + PostgreSQL (dans le compose)
- **Ngrok** (compte + authtoken)
- Un **compte GitHub** (accès au dépôt du projet)

### Prérequis de connaissances

- **Git** : clone, commit, push, notion de webhook
- **Java/Spring Boot** : structure d'un projet, build Maven
- **Notions CI/CD** : stages (build, analyse, déploiement), exécution automatique

##  Architecture

L'application est composée de 4 microservices :
- **car** : Microservice de gestion des voitures
- **client** : Microservice de gestion des clients
- **gateway** : API Gateway (point d'entrée unique)
- **server_eureka** : Eureka Server (service discovery)

Le pipeline CI/CD assure :
- **CI** : Compilation Maven + analyse SonarQube (qualité, code smells, bugs, vulnérabilités)
- **CD** : Déploiement des services dans des conteneurs via Docker Compose
- **Automatisation** : Jenkins exécute le pipeline à chaque push/pull request via webhook GitHub exposé par Ngrok

##  Fichiers du projet

```
TP 32/
├── Jenkinsfile                    # Script de pipeline Jenkins (CI/CD)
├── sonarqube-compose.yml          # Configuration Docker Compose pour SonarQube
├── deploy/
│   └── docker-compose.yml         # Configuration Docker Compose pour les microservices
└── README.md                      # Ce fichier
```

##  Installation et configuration

### Étape 1 : Récupération du projet GitHub

```bash
# Cloner le dépôt
git clone https://github.com/lachgar/jenkins2.git
cd jenkins

# Vérifier la structure
ls
# Doit contenir: car/, client/, gateway/, server_eureka/, deploy/
```

### Étape 2 : Installation et configuration de Jenkins

1. **Installer Jenkins** localement via la [documentation officielle](https://www.jenkins.io/doc/book/installing/)
2. **Configurer le JDK** : Indiquer le chemin du JDK 17 lors de l'installation
3. **Configurer Maven** :
   - Aller dans **Manage Jenkins** → **Tools**
   - Ajouter une installation Maven nommée exactement **`maven`**
   - Choisir installation automatique ou chemin local
4. **Configurer SonarScanner** :
   - Aller dans **Manage Jenkins** → **Tools**
   - Ajouter SonarQube Scanner
   - Installer le plugin SonarQube dans **Manage Jenkins** → **Plugins**

### Étape 3 : Déploiement de SonarQube

```bash
# Démarrer SonarQube avec Docker Compose
docker compose -f sonarqube-compose.yml up -d

# Vérifier que les conteneurs sont démarrés
docker ps

# Accéder à SonarQube (attendre 1-2 minutes pour l'initialisation)
# URL: http://localhost:9999
# Identifiants par défaut: admin/admin
```

**Configuration dans SonarQube :**
1. Créer un projet **`car`** et générer un token (nom: `Car-token`)
2. Créer un projet **`client`** et générer un token (nom: `Client-token`)
3. Noter les tokens générés

**Configuration dans Jenkins :**
1. Aller dans **Manage Jenkins** → **System**
2. Section **SonarQube servers**, ajouter deux serveurs :
   - **Nom** : `SonarQube-Car`, **URL** : `http://localhost:9999`, **Token** : token du projet car
   - **Nom** : `SonarQube-Client`, **URL** : `http://localhost:9999`, **Token** : token du projet client

### Étape 4 : Exposition de Jenkins via Ngrok

```bash
# Installer Ngrok et associer un authtoken (après création de compte)
ngrok config add-authtoken <votre-token>

# Lancer un tunnel HTTP vers Jenkins (port 8080 par défaut)
ngrok http http://localhost:8080

# Copier l'URL publique générée (ex: https://xxxx.ngrok-free.app)
```

**Configuration dans Jenkins :**
1. Aller dans **Manage Jenkins** → **System**
2. Section **GitHub** (ou **GitHub Pull Requests**), ajouter l'URL du dépôt GitHub

**Configuration du Webhook GitHub :**
1. Dans GitHub : **Settings** → **Webhooks** → **Add webhook**
2. **Payload URL** : `https://<URL_NGROK>/github-webhook/`
3. **Content type** : `application/json`
4. **Which events** : `Just the push event`
5. Activer le webhook

### Étape 5 : Création du Job Pipeline Jenkins

1. Dans Jenkins : **New Item** → **Pipeline**
2. Nom du job : `cicd-microservices` (ou autre nom)
3. Dans la configuration :
   - Cocher **GitHub project** et coller l'URL du dépôt
   - Dans **Build Triggers**, cocher **GitHub hook trigger for GITScm polling**
   - Dans **Pipeline** :
     - **Definition** : `Pipeline script`
     - Coller le contenu du fichier **`Jenkinsfile`** fourni
     - Sauvegarder

### Étape 6 : Exécution du pipeline

1. **Lancer un build manuel** : Dans Jenkins, ouvrir le job → **Build Now**
2. **Vérifier les résultats** :
   - Console Output : vérifier que tous les stages passent
   - SonarQube : vérifier les analyses dans http://localhost:9999
   - Docker : vérifier les conteneurs avec `docker ps`
3. **Tester le déclenchement automatique** :
   ```bash
   # Faire une modification et pousser sur GitHub
   git add .
   git commit -m "test: déclenchement webhook"
   git push
   ```
   Jenkins doit démarrer automatiquement une nouvelle exécution.

