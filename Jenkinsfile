pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Backend Build') {
            steps {
                dir('springboot-backend') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Backend Test') {
            steps {
                dir('springboot-backend') {
                    sh 'mvn test'
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    dir('springboot-backend') {
                        sh '''
                            mvn sonar:sonar \
                              -Dsonar.projectKey=employee-management \
                              -Dsonar.projectName=employee-management
                        '''
                    }
                }
            }
        }

    }
}