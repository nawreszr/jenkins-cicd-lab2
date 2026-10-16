# 🚀 Lab Jenkins CI/CD — Spring Boot + Docker + Kubernetes

Projet de laboratoire **DevSecOps** démontrant un **pipeline CI/CD complet** avec Jenkins pour automatiser le déploiement d'une application **Spring Boot** dans **Kubernetes**.

**Auteur** : Nawres Zarrouki

---

## 🎯 Objectifs

- Conteneuriser une application Spring Boot avec Docker
- Pousser l'image sur Docker Hub
- Déployer l'application sur Kubernetes (k3s)
- Automatiser le workflow avec Jenkins Freestyle

## 🛠️ Stack technique

| Composant | Version |
|---|---|
| Java | OpenJDK 21 |
| Maven | 3.6+ (wrapper mvnw) |
| Docker | 29.9.0 |
| Docker Hub | Registry public |
| Kubernetes | k3s 1.36.5 |
| Jenkins | LTS |
| OS | Ubuntu 26.04 Server |

---

## 📦 PARTIE I — Workflow manuel

### Étape 1 — Clone du projet Spring Boot

git clone https://github.com/wahid007/Jenkins-Demo.git
cd Jenkins-Demo

### Étape 2 — Build Maven

chmod +x mvnw
./mvnw clean package -DskipTests

### Étape 3 — Dockerfile

FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/devOpsDemo-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 2222
CMD ["java", "-jar", "app.jar"]

### Étape 4 — Build image Docker

docker build -t nawreszr/jenkins-demo:1.0.0 .

### Étape 5 — Push Docker Hub

docker login -u nawreszr
docker push nawreszr/jenkins-demo:1.0.0

Image : https://hub.docker.com/r/nawreszr/jenkins-demo

![Docker Hub - Partie 1](./docs-lab1/01-docker-hub-partie1.png)

![Docker Hub - Partie 2](./docs-lab1/02-docker-hub-partie2.png)

### Étape 6 — Déploiement Kubernetes

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods
kubectl get svc

![Pods Running](./docs-lab1/05-pods-running.png)

![Pods et Service](./docs-lab1/06-pods-svc.png)

### Étape 7 — Test HTTP

curl http://localhost:30080

![Application dans le navigateur](./docs-lab1/03-app-navigateur.png)

![Test curl](./docs-lab1/04-curl-test.png)

---

## 🔧 PARTIE II — Jenkins Freestyle Workflow

### Dashboard Jenkins

![Dashboard Jenkins](./docs-lab1/07-jenkins-dashboard.png)

### Étape 1 — Job simple freestyle_demo

Script shell :

echo 'Bonjour'
ls -l
echo 'Bonjour DevOps' > test.txt
ls -l

![Config Step 1](./docs-lab1/10-freestyle-step1-config.png)

![Console build 1](./docs-lab1/08-jenkins-build-console.png)

### Étape 2 — Exécution d'un fichier Java

Fichier Test.java :

public class Test {
    public static void main(String[] args) {
        System.out.println("Hi Master DevOps");
    }
}

Commande : java Test.java

![Config Step 2](./docs-lab1/11-freestyle-step2-config.png)

![Console build 2](./docs-lab1/09-jenkins-build-console2.png)

### Étape 3 — Script récupéré depuis GitHub

Dépôt : https://github.com/nawreszr/jenkins-freestyle-demo

Configuration Jenkins :
- Source Code Management : Git
- Repository URL : https://github.com/nawreszr/jenkins-freestyle-demo.git
- Branch : main
- Build Step : chmod +x script.sh && ./script.sh

![Config Step 3-4](./docs-lab1/12-freestyle-step3-4.png)

![Config Step 3-4 détail](./docs-lab1/13-freestyle-step3-4-config.png)

### Étape 4 — Trigger automatique + Rotation

- Déclencheur : Sondage SCM — H/5 * * * *
- Rotation : garder 2 builds max
- Environnement : supprimer l'espace de travail avant le build

![Config Step 5](./docs-lab1/14-freestyle-step5-config.png)

![Build auto GitHub](./docs-lab1/15-trigger-auto-github.png)

---

## 🏆 CHALLENGE — Pipeline complet automatisé

Job : challenge_deploy
Source : https://github.com/wahid007/Jenkins-Demo.git

### Step 1 — Build Maven

chmod +x mvnw
./mvnw clean package -DskipTests
ls -lh target/*.jar

### Step 2 — Build Docker (avec correction du Dockerfile)

Réécriture du Dockerfile pour utiliser eclipse-temurin:17-jre
docker build -t nawreszr/jenkins-demo:latest .

### Step 3 — Push Docker Hub (avec credentials binding)

echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
docker push nawreszr/jenkins-demo:latest

### Step 4 — Déploiement K8s

kubectl set image deployment/spring-app-deployment spring-app-container=nawreszr/jenkins-demo:latest
kubectl rollout restart deployment/spring-app-deployment
kubectl rollout status deployment/spring-app-deployment --timeout=180s

### Step 5 — Vérification

kubectl get pods
kubectl get svc
curl -s http://localhost:30080 | head -5

![Challenge 5 steps](./docs-lab1/16-challenge-5-steps.png)

![Challenge SUCCESS](./docs-lab1/17-challenge-success.png)

---

## 🏗️ Architecture du pipeline

GitHub (source) → Jenkins → Maven Build → Docker Build → Docker Hub → Kubernetes Deploy → Test HTTP

---

## 🎓 Compétences mises en œuvre

- Docker : Dockerfile custom, build, tag, push
- Docker Hub : registry, PAT, credentials
- Kubernetes : Deployment, Service NodePort, replicas, rolling update
- Jenkins : freestyle jobs, SCM Git, credentials binding
- CI/CD : pipeline automatisé GitHub → Registry → K8s
- Troubleshooting : disk pressure, permissions Docker, images dépréciées

---

## 🐛 Problèmes résolus

| Problème | Solution |
|---|---|
| openjdk:17-jdk-slim introuvable | Remplacé par eclipse-temurin:17-jre |
| disk-pressure sur k3s | Nettoyage disque + eviction-hard |
| Jenkins ne voit pas Docker | usermod -aG docker jenkins |
| kubectl pointe vers Jenkins | Copie du kubeconfig k3s |
| Mot de passe GitHub refusé | Utilisation de SSH (clé ed25519) |
| Maven extrêmement lent | Miroir maven.aliyun.com |

---

## 👤 Auteur

**Nawres Zarrouki** — DevSecOps

- GitHub : https://github.com/nawreszr
- Docker Hub : https://hub.docker.com/u/nawreszr

---

⭐ Projet réalisé dans le cadre d'une formation DevSecOps (formateur : Wahid Hamdi).
# jenkins-cicd-lab2
# nawreszr-jenkins-cicd-lab2
