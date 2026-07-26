pipeline {
    agent any

    environment {
        IMAGE_NAME = "anwayalamwar/todo_app_sunbeam-web"
        IMAGE_TAG  = "latest"
        NAMESPACE  = "secureflow"
        APP_URL    = "http://192.168.100.20:30080"     // Change to your application's URL
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('PHP Validation') {
            steps {
                sh '''
                echo "Validating PHP files..."
                find . -name "*.php" -exec php -l {} \\;
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    /opt/sonar-scanner/bin/sonar-scanner
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Trivy Image Scan') {
    		steps {
        	sh '''
        	rm -rf /tmp/trivy-cache

        	trivy image \
          	--cache-dir /tmp/trivy-cache \
          	--severity HIGH,CRITICAL \
          	--format table \
         	 --output trivy-report.txt \
        	  ${IMAGE_NAME}:${IMAGE_TAG}
       	 	'''
    		}
	}	

        stage('Archive Trivy Report') {
            steps {
                archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker push ${IMAGE_NAME}:${IMAGE_TAG}

                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/namespace.yaml
                kubectl apply -f k8s/secret.yaml
                kubectl apply -f k8s/configmap.yaml
                kubectl apply -f k8s/mysql-pv.yaml
                kubectl apply -f k8s/mysql-pvc.yaml
                kubectl apply -f k8s/mysql-service.yaml
                kubectl apply -f k8s/mysql-deployment.yaml
                kubectl apply -f k8s/todo-service.yaml
                kubectl apply -f k8s/todo-deployment.yaml
                '''
            }
        }

        stage('Wait for Deployment') {
            steps {
                sh '''
                kubectl rollout status deployment/todo-app -n ${NAMESPACE}
                kubectl get pods -n ${NAMESPACE}
                '''
            }
        }

        stage('OWASP ZAP Baseline Scan') {
            steps {
                sh '''
                docker run --rm \
                -v $(pwd):/zap/wrk/:rw \
                ghcr.io/zaproxy/zaproxy:stable \
                zap-baseline.py \
                -t ${APP_URL} \
                -r zap-report.html
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

        success {
            echo "======================================"
            echo "SecureFlow Pipeline Completed Successfully"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "Pipeline Failed"
            echo "Check the failed stage."
            echo "======================================"
        }

        always {
            cleanWs()
        }
    }
}
