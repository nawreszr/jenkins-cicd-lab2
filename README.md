# Lab 2 – Pipeline CI/CD avec Jenkins, Maven et Docker

Mise en place d'un pipeline CI/CD avec Jenkins qui clone un projet Spring Boot, le compile avec Maven, construit une image Docker et déploie le conteneur.

> **Enseignant :** Wahid Hamdi
> **Projet de base :** <https://github.com/wahid007/Jenkins-Demo>

---

## Sommaire

1. [Architecture du pipeline](#1-architecture-du-pipeline)
2. [Prérequis](#2-prérequis)
3. [Installation de Java et Jenkins](#3-installation-de-java-et-jenkins)
4. [Installation des plugins](#4-installation-des-plugins)
5. [Configuration des outils (JDK, Maven)](#5-configuration-des-outils-jdk-maven)
6. [Jenkinsfile](#6-jenkinsfile)
7. [Dockerfile](#7-dockerfile)
8. [Exécution du pipeline](#8-exécution-du-pipeline)
9. [Vérification du déploiement](#9-vérification-du-déploiement)
10. [Problèmes rencontrés](#10-problèmes-rencontrés)

---

## 1. Architecture du pipeline

```
Start → Clone Repo → Build Project → Build Docker Image → Deploy Docker Container → Test HTTP → End
```

| Étape | Rôle |
|---|---|
| Clone Repo | Récupère le code depuis GitHub |
| Build Project | `mvn clean package` pour générer le `.jar` |
| Build Docker Image | Construit l'image à partir du `Dockerfile` |
| Deploy Docker Container | Lance le conteneur sur le port `2222` |
| Test HTTP | Vérifie avec `curl` que l'application répond |

---

## 2. Prérequis

- VM Ubuntu Server (4 Go de RAM minimum, 50 Go d'espace disque)
- Docker installé
- Kubernetes (K3S) installé
- Un compte GitHub (fork du projet)

```bash
docker --version
java -version
```

---

## 3. Installation de Java et Jenkins

### Installer Java

```bash
sudo apt update
sudo apt install -y fontconfig openjdk-21-jdk
java -version
```

> Installer le paquet **`openjdk-21-jdk`** (et non seulement `-jre`) : Jenkins a besoin d'un vrai JDK pour la configuration des outils (voir section 10).

### Installer Jenkins (service sur Ubuntu)

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

### Accéder à Jenkins

Ouvrir `http://HOST-IP:8080/`, puis récupérer le mot de passe initial :

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Choisir **Installer les plugins suggérés**, créer l'administrateur et confirmer l'URL de Jenkins.

### Autoriser Jenkins à utiliser Docker

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## 4. Installation des plugins

**Administrer Jenkins → Plugins → Plugins disponibles**, installer :

- Kubernetes CLI
- Maven Integration
- Docker (et Docker Pipeline)

![Progression du téléchargement des plugins](screenshots/01-plugins-installation.png)

Tous les plugins (Kubernetes Client API, Kubernetes CLI, Maven Integration, Docker, Docker Commons, Docker API, Docker Pipeline…) sont installés avec le statut **Succès**.

---

## 5. Configuration des outils (JDK, Maven)

**Administrer Jenkins → Tools (Configuration globale des outils)**

### JDK

Sous Linux, il suffit d'indiquer le chemin `JAVA_HOME` du JDK installé :

```bash
ls /usr/lib/jvm/
# java-21-openjdk-amd64
```

- **Nom :** `JDK-21`
- **JAVA_HOME :** `/usr/lib/jvm/java-21-openjdk-amd64`
- *Install automatically* : décoché

![Configuration du JDK](screenshots/02-config-jdk.png)

### Maven

- **Nom :** `Maven-3.9`
- *Install automatically* : coché → *Install from Apache*

![Configuration de Maven](screenshots/03-config-maven.png)

> Le nom donné ici (`Maven-3.9`, `JDK-21`) doit correspondre exactement à celui utilisé dans le `Jenkinsfile`.

---

## 6. Jenkinsfile

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk   'JDK-21'
    }

    environment {
        IMAGE_NAME = "nawreszr/jenkins-demo"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/nawreszr/Jenkins-Demo.git'
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy Docker Container') {
            steps {
                sh "docker rm -f devopsexample || true"
                sh "docker run --name devopsexample -d -p 2222:2222 ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Test HTTP') {
            steps {
                echo '🧪 Test HTTP...'
                sh 'curl -s http://localhost:2222 | head -5 || true'
                echo '✅ Pipeline terminé avec succès !'
            }
        }
    }
}
```

---

## 7. Dockerfile

```dockerfile
FROM openjdk:17-jdk-slim
VOLUME /tmp
COPY target/devOpsDemo-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
EXPOSE 2222
```

---

## 8. Exécution du pipeline

Dans Jenkins : ouvrir le pipeline `Jenkins_test` puis cliquer sur **Lancer un build**.

### Vue du pipeline

Toutes les étapes sont au vert : *Clone Repo*, *Build Project*, *Build Docker Image*, *Deploy Docker Container* et *Test HTTP*.

![Vue du pipeline Jenkins](screenshots/04-pipeline-stages.png)

### Journal de la console

Le build se termine avec `Finished: SUCCESS`.

![Fin du pipeline - SUCCESS](screenshots/05-pipeline-success-log.png)

---

## 9. Vérification du déploiement

### Côté serveur (commande Linux)

```bash
docker ps | grep devopsexample
```

Le conteneur `devopsexample` est démarré (image `nawreszr/jenkins-demo:3`) et le port `2222` est exposé :

![docker ps](screenshots/06-docker-ps.png)

### Test en ligne de commande

```bash
curl -s http://localhost:2222 | head -5
```

### Côté navigateur

L'application est accessible à l'adresse `http://192.168.42.130:2222` :

![Application déployée](screenshots/07-application-navigateur.png)

---

## 10. Problèmes rencontrés

| Problème | Cause | Solution |
|---|---|---|
| `/usr/lib/jvm/java-21-openjdk-amd64 ne semble pas être un répertoire contenant un JDK` | Seul le JRE était installé, sans `javac` | `sudo apt install openjdk-21-jdk` puis vérifier `ls /usr/lib/jvm/java-21-openjdk-amd64/bin/javac` |
| `docker: permission denied` dans le pipeline | L'utilisateur `jenkins` n'est pas dans le groupe `docker` | `sudo usermod -aG docker jenkins && sudo systemctl restart jenkins` |
| `container name "devopsexample" is already in use` | Le conteneur d'un build précédent tourne encore | `docker rm -f devopsexample` avant `docker run` |
| Nom d'outil introuvable (`tool 'Maven-3.9'`) | Le nom du `Jenkinsfile` diffère de celui configuré dans Jenkins | Utiliser exactement le même nom |

---

## Commandes utiles

```bash
sudo systemctl status jenkins
sudo journalctl -u jenkins -f

docker ps
docker images
docker logs devopsexample
docker rm -f devopsexample
```

## Auteur

**Nawres Zarrouki** – Master DevOps & Cloud
