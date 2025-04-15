## Déploiement CI/CD avec Jenkins

### Structure du dépôt

Créez un dépôt Git et placez-y les fichiers suivants :

- `Dockerfile` : Fichier pour construire l’image Docker avec WordPress personnalisé.
- `deployment.yml` : Fichier Kubernetes pour déployer l’application.
- `service.yml` : Fichier Kubernetes pour exposer le service via NodePort.
- `Jenkinsfile` : Pipeline Jenkins de build, push et déploiement.

---

### Configuration de Jenkins

1. **Créer une credential Docker Hub dans Jenkins :**
   - Aller dans : `Jenkins > Manage Jenkins > Credentials`
   - Ajouter une nouvelle credential :
     - **Type** : `Username and password`
     - **ID** : `dockerhub_credentials`
     - **Username** : votre identifiant Docker Hub
     - **Password** : votre mot de passe ou token Docker Hub

2. **Configurer un job Jenkins Pipeline :**
   - Choisir **Pipeline project**
   - Dans la section **Pipeline > Definition**, sélectionner `Pipeline script from SCM`
   - SCM : `Git`
   - Renseigner l'URL du dépôt GitHub (ex. `https://github.com/nicholasevrard/dentiste`)
   - Jenkins détectera automatiquement les modifications et lancera le pipeline (`Jenkinsfile`)

---

### Accès à Kubernetes (kubectl)

Assurez-vous que Jenkins a accès à votre cluster Kubernetes (Minikube, MicroK8s, etc.) :

- Le conteneur Jenkins doit pouvoir exécuter `kubectl`
- Le fichier `~/.kube/config` doit être **copié ou monté** dans le conteneur Jenkins (volume partagé)
- Testez dans Jenkins :  
  ```bash
  kubectl get nodes

