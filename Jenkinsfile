```groovy
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
                    sh 'mvn clean verify'
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
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package JAR') {
            steps {
                dir('springboot-backend') {
                    sh 'mvn package -DskipTests'
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
```
