pipeline {
    agent any

    environment {
        IMAGE_NAME = "anwayalamwar/todo_app_sunbeam-web"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('PHP Syntax Check') {
            steps {
                sh 'find . -name "*.php" -exec php -l {} \\;'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image --exit-code 0 $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push $IMAGE_NAME:$IMAGE_TAG
                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl rollout status deployment/todo-app -n secureflow
                kubectl get pods -n secureflow
                '''
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f'
        }
    }
}
