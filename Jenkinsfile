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

    }
}
