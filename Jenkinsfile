pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nicholasevrard/dentist2"
        DOCKER_TAG = "latest"
        REGISTRY_CREDENTIALS = 'dockerhub_credentials'
        SONARQUBE_ENV = 'SonarQube'
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/nicholasevrard/dentist2.git', branch: 'main'
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'sonar-scanner -Dsonar.projectKey=dentist2 -Dsonar.sources=. -Dsonar.php.file.suffixes=.php -Dsonar.host.url=http://localhost:9000'
                }
            }
        }

        stage('Analyse Trivy (Image scan)') {
            steps {
                sh '''
                    trivy image --format table --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                '''
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                sh '''
                    mkdir -p owasp-report
                    dependency-check --project "Dentist2" --scan . --format "HTML" --out owasp-report || true
                '''
                archiveArtifacts artifacts: 'owasp-report/dependency-check-report.html', allowEmptyArchive: true
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${REGISTRY_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
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

    post {
        success {
            echo "Pipeline terminé avec succès"
        }
        failure {
            echo "Le pipeline a échoué"
        }
    }
}
