## Déploiement CI/CD avec Jenkins, Docker, Kubernetes et SonarQube

Ce dépôt contient :

- `Dockerfile` – image Ubuntu/Apache/WordPress
- `deployment.yml` – déploiement Kubernetes
- `service.yml` – service NodePort pour exposer l'application
- `Jenkinsfile` – pipeline CI/CD avec SonarQube, Docker, Kubernetes

---

### Configuration Jenkins

#### 1. Repo GitHub

Configurez un job Jenkins qui pointe vers ce repo :  
 [`https://github.com/nicholasevrard/dentist2`](https://github.com/nicholasevrard/dentist2)

Le pipeline s'exécutera automatiquement à chaque commit (grâce à `pollSCM`).
   - Choisir **Pipeline project**
   - Dans la section **Pipeline > Definition**, sélectionner `Pipeline script from SCM`
   - SCM : `Git`
   - Renseigner l'URL du dépôt GitHub (ex. `https://github.com/nicholasevrard/dentiste`)
   - Jenkins détectera automatiquement les modifications et lancera le pipeline (`Jenkinsfile`)
---

#### 2. Credentials DockerHub

Créez une **credential** Docker Hub :

- **ID** : `dockerhub_credentials`
- **Type** : Username + Password
   - Aller dans : `Jenkins > Manage Jenkins > Credentials`
   - Ajouter une nouvelle credential :
     - **Type** : `Username and password`
     - **ID** : `dockerhub_credentials`
     - **Username** : votre identifiant Docker Hub
     - **Password** : votre mot de passe ou token Docker Hub
---

#### 3. Accès Kubernetes

Vérifiez que Jenkins peut exécuter `kubectl` :  
Le fichier `~/.kube/config` doit être **présent dans le conteneur Jenkins** (soit monté depuis la VM, soit copié).
Assurez-vous que Jenkins a accès à votre cluster Kubernetes (Minikube, MicroK8s, etc.) :

- Le conteneur Jenkins doit pouvoir exécuter `kubectl`
- Le fichier `~/.kube/config` doit être **copié ou monté** dans le conteneur Jenkins (volume partagé)
- Testez dans Jenkins :  
  ```bash
  kubectl get nodes
---

#### 4. Intégration SonarQube

- Configurez SonarQube dans Jenkins (`Manage Jenkins > Configure System`)
- Ajoutez un token dans `SonarQube servers`
- Ajoutez `sonar-scanner` dans les outils (`Global Tool Configuration`)
- Le fichier `sonar-project.properties` est requis à la racine du repo

> L’analyse statique du code PHP se fait automatiquement en début de pipeline.

---

Un pipeline DevOps complet de build, test, push, analyse, et déploiement.

---

## Sécurité DevSecOps intégrée

Ce pipeline CI/CD intègre plusieurs outils de sécurité :

- **Trivy** : Scanne les images Docker pour détecter les vulnérabilités connues
- **OWASP Dependency-Check** : Analyse les dépendances PHP à la recherche de failles connues.
- **SonarQube** : Analyse statique du code source (qualité, duplication, bugs).

### Résultats disponibles :
- Les rapports OWASP sont générés dans `dependency-check-report/`.
- Les logs Trivy s'affichent dans la console Jenkins.
- Les rapports SonarQube sont consultables depuis l'interface Sonar.

---

![Build Status](http://192.168.8.121:8085/job/dentist2/badge/icon)

## 🚀 Pipeline CI/CD avec Jenkins

Ce projet utilise un pipeline automatisé avec **Jenkins** pour effectuer l'intégration et le déploiement continus (CI/CD) de l'application WordPress personnalisée. Le pipeline est défini dans un fichier `Jenkinsfile` et inclut plusieurs étapes pour garantir la qualité, la sécurité et la livraison fluide de l'application dans un cluster Kubernetes.

### 🔧 Étapes du pipeline :

| Étape                    | Description |
|--------------------------|-------------|
| **Checkout**           | Clonage du dépôt GitHub contenant le Dockerfile, Jenkinsfile et les fichiers Kubernetes (`deployment.yml`, `service.yml`). |
| **Analyse SonarQube** | Utilise SonarQube pour analyser le code PHP et détecter les bugs, les vulnérabilités, la dette technique et les mauvaises pratiques. |
| **Scan Trivy**        | Lance un scan de sécurité avec **Trivy** pour analyser l’image Docker générée et détecter les vulnérabilités connues (CVEs). |
| **Scan OWASP (ZAP)**  | Effectue un test de pénétration automatisé avec **OWASP ZAP** pour identifier les failles de sécurité potentielles dans l'application Web. |
| **Build Docker Image**| Crée une image Docker personnalisée à partir du Dockerfile. |
| **Push Docker Image** | Pousse l’image sur Docker Hub à l’aide d’une credential sécurisée (`dockerhub_credentials`). |
| **Déploiement K8s**    | Déploie l’application dans le cluster Kubernetes (via `deployment.yml` et `service.yml`). |

### Prérequis Jenkins :

- Une **credential Docker Hub** dans Jenkins :
  - **ID** : `dockerhub_credentials`
  - **Type** : Username and password

- Jenkins doit avoir accès à :
  - Docker (`/var/run/docker.sock`)
  - Kubernetes (`~/.kube/config`)
  - SonarQube via l'URL et le token
  - Trivy (installé sur la VM ou dans l'image Jenkins)
  - OWASP ZAP (installé ou conteneurisé)

---

Ce pipeline garantit que **chaque modification dans le code déclenche automatiquement une série de tests de qualité et de sécurité**, suivis du déploiement dans un environnement Kubernetes. Cela assure une livraison continue fiable et conforme aux bonnes pratiques DevSecOps.
