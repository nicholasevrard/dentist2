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
