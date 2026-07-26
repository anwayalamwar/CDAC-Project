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
        	-f table \
        	-o trivy-report.txt \
        	anwayalamwar/todo_app_sunbeam-web:latest
        	'''
    	}
	stage('Archive Trivy Report') {
	    steps {
       	archiveArtifacts artifacts: 'trivy-report.txt', finge		rprint: true
		    }
		}
	}
    }
}
