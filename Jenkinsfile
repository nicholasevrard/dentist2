pipeline {
    agent any

    triggers {
        pollSCM('*/1 * * * *')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    environment {
        DOCKER_IMAGE_NAME = 'nicholasevrard/dentist2'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Scan OWASP Dependencies') {
            steps {
                sh '''
                    dependency-check.sh --project "dentist2" --scan . --format "HTML" --out dependency-check-report
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Scan Trivy (Docker Image)') {
            steps {
                sh '''
                    trivy image --exit-code 0 --severity HIGH,CRITICAL $DOCKER_IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Push Docker Image') {
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

        stage('Déploiement Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                '''
            }
        }
    }
}
