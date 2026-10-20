# 🚀 Lab 2 — Pipeline CI/CD avec Jenkins & Spring Boot

Ce laboratoire documente la mise en place d'un **pipeline CI/CD complet** avec **Jenkins** pour automatiser le build, la conteneurisation et le déploiement d'une application **Spring Boot** (Java/Maven).

**Auteur** : Nawres Zarrouki

---

## 🎯 Objectifs

- **Forker** un projet Spring Boot sur GitHub.
- **Installer** les plugins Jenkins (Maven, Docker, Git).
- **Configurer** les outils globaux (JDK, Maven).
- **Créer un Jenkinsfile** (pipeline déclaratif).
- **Automatiser** : Clone → Build → Docker → Deploy → Test.
- **Valider** l'exécution complète avec `Finished: SUCCESS`.

---

## 🛠️ Stack Technique

| Composant | Version / Détail |
|---|---|
| **Jenkins** | LTS (service systemd) |
| **Java** | OpenJDK 21 (`/usr/lib/jvm/java-21-openjdk-amd64`) |
| **Maven** | 3.9 (installation automatique depuis Apache) |
| **Docker** | Plugin Jenkins + Moteur Docker natif |
| **Git / GitHub** | Versioning & SCM |
| **OS** | Ubuntu Server 26.04 |

---

## 📦 Étapes du Laboratoire

### Étape 1 — Fork du Projet & Plugins

Le projet de base `https://github.com/wahid007/Jenkins-Demo` a été forké vers :
🔗 **[https://github.com/nawreszr/Jenkins-Demo](https://github.com/nawreszr/Jenkins-Demo)**

**Plugins installés :**
- Kubernetes CLI
- Maven Integration
- Docker & Docker Pipeline

> **📸 Preuve :** Installation réussie des plugins.

![Plugins installés](![Plugins installés](https://raw.githubusercontent.com/nawreszr/jenkins-cicd-lab2/main/docs/01-plugins-installed.png))

### Étape 2 — Configuration Globale des Outils

Dans **Administrer Jenkins → Configuration globale des outils** :

1.  **JDK** :
    - **Nom** : `JDK-21`
    - **JAVA_HOME** : `/usr/lib/jvm/java-21-openjdk-amd64`

2.  **Maven** :
    - **Nom** : `Maven-3.9`
    - **Install automatically** : ✅ (depuis Apache, version 3.9)

### Étape 3 — Jenkinsfile

Le fichier `Jenkinsfile` (à la racine du dépôt) définit les 5 étapes du pipeline :

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk 'JDK-21'
    }

    environment {
        DOCKER_IMAGE_TAG = "nawreszr/jenkins-demo:${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/nawreszr/Jenkins-Demo.git'
            }
        }
        stage('Build Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                    cat > Dockerfile << 'EOF'
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/devOpsDemo-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 2222
CMD ["java", "-jar", "app.jar"]
EOF
                    docker build -t ${DOCKER_IMAGE_TAG} .
                '''
            }
        }
        stage('Deploy Docker Container') {
            steps {
                sh '''
                    docker rm -f devopsexample || true
                    docker run --name devopsexample -d -p 2222:2222 ${DOCKER_IMAGE_TAG}
                '''
            }
        }
        stage('Test HTTP') {
            steps {
                sh 'curl -s http://localhost:2222 | head -5'
            }
        }
    }
}
Étape 4 — Création & Exécution du Job
Dans Jenkins, un job Pipeline nommé Jenkins_test a été créé avec la configuration :

Definition : Pipeline script from SCM

SCM : Git

Repository URL : https://github.com/nawreszr/Jenkins-Demo.git

Script Path : Jenkinsfile

🚀 Résultats & Validation
1. Pipeline : 5 Stages en SUCCESS
Toutes les étapes (Clone, Build Maven, Build Docker, Deploy, Test HTTP) se sont exécutées avec succès.

📸 Preuve : Vue "Stage View" avec les 5 étapes vertes.

![Pipeline stages](https://raw.githubusercontent.com/nawreszr/jenkins-cicd-lab2/main/docs/04-pipeline-success.png)

2. Console Output : Finished: SUCCESS
La console Jenkins confirme la fin du pipeline sans erreur.

📸 Preuve : Log final Finished: SUCCESS.

![Console output](https://raw.githubusercontent.com/nawreszr/jenkins-cicd-lab2/main/docs/05-pipeline-console.png)

3. Conteneur Docker en Exécution
L'image Docker a été construite et le conteneur devopsexample tourne sur le port 2222.

📸 Preuve : docker ps montrant le conteneur actif.

![Docker container](https://raw.githubusercontent.com/nawreszr/jenkins-cicd-lab2/main/docs/06-docker-container.png)

4. Application Accessible
L'application Spring Boot répond correctement sur http://192.168.78.129:2222.

📸 Preuve : Page d'accueil Spring Boot dans le navigateur.

![Application](https://raw.githubusercontent.com/nawreszr/jenkins-cicd-lab2/main/docs/07-app-navigateur.png)

🐛 Problèmes Rencontrés & Solutions
Problème	Solution
JDK Path Error : Jenkins ne reconnaissait pas le chemin JAVA_HOME.	Vérification du chemin réel via readlink -f $(which java).
Maven Lent : Téléchargement des dépendances très long.	Configuration d'un miroir Aliyun (maven.aliyun.com).
Docker Permission : Jenkins ne pouvait pas exécuter Docker.	Ajout de l'utilisateur jenkins au groupe docker + redémarrage.
🎓 Compétences Validées
✅ Jenkins : Installation, gestion des plugins, configuration des outils.

✅ Pipeline as Code : Écriture d'un Jenkinsfile déclaratif.

✅ Maven : Build automatisé d'une application Spring Boot.

✅ Docker : Construction d'image et déploiement de conteneur depuis Jenkins.

✅ Troubleshooting : Résolution de problèmes de chemins, de réseau et de permissions.

👤 Auteur
Nawres Zarrouki — Étudiant DevSecOps

GitHub : @nawreszr

Docker Hub : @nawreszr

⭐ Laboratoire réalisé dans le cadre d'une formation DevSecOps.
