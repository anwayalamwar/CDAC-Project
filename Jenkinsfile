pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('PHP Validation') {
            steps {
                sh '''
                find . -name "*.php" -exec php -l {} \\;
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '/opt/sonar-scanner/bin/sonar-scanner'
                }
            }
        }
	stage('Trivy Image Scan') {
    		steps {
        		sh '''
        	trivy image \
        	--format template \
        	--template "@$HOME/.trivy/html.tpl" \
        	-o trivy-report.html \
        	anwayalamwar/todo_app_sunbeam-web:latest
        	'''
    		}
	}
	stage('Publish Trivy Report') {
	    steps {
        	archiveArtifacts artifacts: 'trivy-report.htm			l', fingerprint: true
    		}
	}
    }
}
