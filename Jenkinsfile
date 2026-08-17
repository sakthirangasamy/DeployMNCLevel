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

        stage('Build & Test') {
            steps {
                dir('springboot-backend') {
                    sh 'mvn clean verify -DforkCount=0'
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                dir('springboot-backend') {

                    withSonarQubeEnv('SonarQube') {

                        sh '''
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.4.0.6343:sonar \
                              -Dsonar.projectKey=employee-management \
                              -Dsonar.projectName=employee-management
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {

                timeout(time: 10, unit: 'MINUTES') {

                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package JAR') {
            steps {

                dir('springboot-backend') {

                    sh 'mvn package -DskipTests'

                    sh 'ls -lh target/*.jar'
                }
            }
        }
    }

    post {

        success {
            echo 'Phase 1 completed successfully - JAR generated!'
        }

        failure {
            echo 'Phase 1 failed!'
        }
    }
}