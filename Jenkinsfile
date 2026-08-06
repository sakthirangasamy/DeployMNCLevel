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
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.2.0.4988:sonar \
                              -Dsonar.projectKey=employee-management \
                              -Dsonar.projectName=employee-management
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package JAR') {
            steps {
                dir('springboot-backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('springboot-backend') {
                    sh 'docker build -t employee-management:1.0 .'
                }
            }
        }

    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}