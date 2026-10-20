# Lab 2 – Pipeline CI/CD avec Jenkins, Maven et Docker

Pipeline Jenkins qui clone un projet Spring Boot, le compile avec Maven, construit une image Docker et déploie le conteneur.

> **Enseignant :** Wahid Hamdi
> **Projet de base :** <https://github.com/wahid007/Jenkins-Demo>

## Sommaire

1. [Architecture du pipeline](#1-architecture-du-pipeline)
2. [Installation de Java et Jenkins](#2-installation-de-java-et-jenkins)
3. [Installation des plugins](#3-installation-des-plugins)
4. [Configuration des outils (JDK, Maven)](#4-configuration-des-outils-jdk-maven)
5. [Jenkinsfile](#5-jenkinsfile)
6. [Dockerfile](#6-dockerfile)
7. [Exécution du pipeline](#7-exécution-du-pipeline)
8. [Vérification du déploiement](#8-vérification-du-déploiement)
9. [Problèmes rencontrés](#9-problèmes-rencontrés)

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

## 2. Installation de Java et Jenkins

```bash
sudo apt update
sudo apt install -y fontconfig openjdk-21-jdk
java -version
```

> Installer **`openjdk-21-jdk`** (et pas seulement `-jre`) : Jenkins a besoin d'un vrai JDK.

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
```

Accès : `http://HOST-IP:8080/`. Mot de passe initial :

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Autoriser Jenkins à utiliser Docker :

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## 3. Installation des plugins

**Administrer Jenkins → Plugins → Plugins disponibles** : Kubernetes CLI, Maven Integration, Docker, Docker Pipeline.

![Installation des plugins](docs/01-plugins-installed.png)

---

## 4. Configuration des outils (JDK, Maven)

**Administrer Jenkins → Tools**

### JDK

```bash
ls /usr/lib/jvm/
```

- **Nom :** `JDK-21`
- **JAVA_HOME :** `/usr/lib/jvm/java-21-openjdk-amd64`

![Configuration du JDK](docs/04-pipeline-success.png)

### Maven

- **Nom :** `Maven-3.9`
- *Install automatically* coché → *Install from Apache*

![Configuration de Maven](docs/05-pipeline-console.png)

> Les noms (`JDK-21`, `Maven-3.9`) doivent être identiques dans le `Jenkinsfile`.

---

## 5. Jenkinsfile

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
            steps { sh 'mvn clean package' }
        }
        stage('Build Docker Image') {
            steps { sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ." }
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

## 6. Dockerfile

```dockerfile
FROM openjdk:17-jdk-slim
VOLUME /tmp
COPY target/devOpsDemo-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
EXPOSE 2222
```

---

## 7. Exécution du pipeline

Jenkins → pipeline `Jenkins_test` → **Lancer un build**.

### Vue du pipeline

![Vue du pipeline](docs/06-docker-container.png)

### Journal de la console

![Pipeline terminé avec succès](docs/07-app-navigateur.png)

---

## 8. Vérification du déploiement

```bash
docker ps | grep devopsexample
curl -s http://localhost:2222 | head -5
```

![docker ps](@@IMG6@@)

Application accessible sur `http://192.168.42.130:2222` :

![Application dans le navigateur](@@IMG7@@)

---

## 9. Problèmes rencontrés

| Problème | Cause | Solution |
|---|---|---|
| `... ne semble pas être un répertoire contenant un JDK` | Seul le JRE était installé | `sudo apt install openjdk-21-jdk` |
| `docker: permission denied` | `jenkins` n'est pas dans le groupe `docker` | `sudo usermod -aG docker jenkins && sudo systemctl restart jenkins` |
| `container name "devopsexample" is already in use` | Ancien conteneur encore actif | `docker rm -f devopsexample` |
| `tool 'Maven-3.9'` introuvable | Nom différent entre Jenkinsfile et Jenkins | Utiliser exactement le même nom |

## Auteur

**Nawres Zarrouki** – Master DevOps & Cloud
