pipeline {
    agent any

    environment {
        IMAGE_NAME = "anwayalamwar/todo_app_sunbeam-web"
        IMAGE_TAG  = "${BUILD_NUMBER}"

        SECURITY_VM = "192.168.100.30"
        SECURITY_USER = "sunbeam"

        APP_URL = "http://192.168.100.21:30080"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('PHP Syntax Check') {
            steps {
                sh '''
                find . -name "*.php" -exec php -l {} \\;
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    /opt/sonar-scanner/bin/sonar-scanner
                    '''
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
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh '''
                    echo "$PASS" | docker login -u "$USER" --password-stdin
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

        stage('OWASP ZAP Scan') {

                sh '''
                ssh -o StrictHostKeyChecking=no ${SECURITY_USER}@${SECURITY_VM} \
                "mkdir -p ~/zap-reports && \
		rm -f ~/zap-reports/zap-report.html && \

                docker run --rm \
                  -v ~/zap-reports:/zap/wrk:Z \
                  ghcr.io/zaproxy/zaproxy:stable \
                  zap-baseline.py \
                  -t ${APP_URL} \
                  -r zap-report.html"
                '''
            }
        }

        stage('Collect ZAP Report') {
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no \
                ${SECURITY_USER}@${SECURITY_VM}:~/zap-reports/zap-report.html \
                ${WORKSPACE}/
                '''
            }
        }

        stage('Archive ZAP Report') {
            steps {
                archiveArtifacts artifacts: 'zap-report.html', fingerprint: true
            }
        }

    }

    post {

        always {

            sh '''
            docker image prune -f
            '''

            cleanWs()
        }

        success {
            echo 'SecureFlow Pipeline Completed Successfully.'
        }

        failure {
            echo 'SecureFlow Pipeline Failed.'
        }
    }
}
