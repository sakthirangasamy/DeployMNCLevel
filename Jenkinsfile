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
                    sh 'mvn clean compile -Dspring.profiles.active=docker'
                }
            }
        }


        stage('Backend Test') {
            steps {
                dir('springboot-backend') {
                    sh 'mvn test -Dspring.profiles.active=docker'
                }
            }
        }


        stage('Frontend Install') {
            steps {
                dir('react-frontend') {
                    sh 'npm install'
                }
            }
        }


        stage('Frontend Build') {
            steps {
                dir('react-frontend') {
                    sh 'npm run build'
                }
            }
        }


        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {

                    dir('springboot-backend') {

                        sh '''
                        mvn sonar:sonar \
                        -Dspring.profiles.active=docker \
                        -Dsonar.projectKey=employee-management \
                        -Dsonar.projectName=employee-management
                        '''

                    }
                }
            }
        }

    }
}