pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'nicholasevrard/dentiste2'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git 'https://github.com/nicholasevrard/dentiste2.git'
            }
        }

        stage('Construire l’image Docker') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Pousser sur Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE_NAME:$IMAGE_TAG
                        docker logout
                    '''
                }
            }
        }

        stage('Déployer sur Kubernetes (Minikube)') {
            steps {
                sh '''
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                '''
            }
        }
    }

    post {
        success {
            echo "Déploiement réussi sur Kubernetes !"
        }
        failure {
            echo "Échec du pipeline"
        }
    }
}
