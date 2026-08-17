pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test GitHub Connection') {
            steps {
                sh 'echo "GitHub connected successfully!"'
                sh 'git log -1 --oneline'
            }
        }
    }
}