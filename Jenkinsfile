// PHASE -1
// pipeline {

//     agent any

//     tools {
//         jdk 'JDK17'
//         maven 'Maven3'
//     }

//     stages {

//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Build & Test') {
//             steps {
//                 dir('springboot-backend') {
//                     sh 'mvn clean verify -DforkCount=0'
//                 }
//             }
//         }

//         stage('SonarQube Scan') {
//             steps {
//                 dir('springboot-backend') {

//                     withSonarQubeEnv('SonarQube') {

//                         sh '''
//                             mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.4.0.6343:sonar \
//                               -Dsonar.projectKey=employee-management \
//                               -Dsonar.projectName=employee-management
//                         '''
//                     }
//                 }
//             }
//         }

//         stage('Quality Gate') {
//             steps {

//                 timeout(time: 10, unit: 'MINUTES') {

//                     waitForQualityGate abortPipeline: true
//                 }
//             }
//         }

//         stage('Package JAR') {
//             steps {

//                 dir('springboot-backend') {

//                     sh 'mvn package -DskipTests'

//                     sh 'ls -lh target/*.jar'
//                 }
//             }
//         }
//     }

//     post {

//         success {
//             echo 'Phase 1 completed successfully - JAR generated!'
//         }

//         failure {
//             echo 'Phase 1 failed!'
//         }
//     }
// }

// PHASE -2

pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        IMAGE_NAME = 'employee-management'
        IMAGE_TAG = "${BUILD_NUMBER}"
        NEXUS_REGISTRY = 'localhost:8082'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Spring Boot') {
            steps {
                dir('springboot-backend') {
                    sh '''
                        echo "Java version:"
                        java -version

                        echo "Maven version:"
                        mvn -version

                        echo "Building Spring Boot application..."
                        mvn clean package -DskipTests
                    '''
                }
            }
        }

        stage('Verify JAR') {
            steps {
                dir('springboot-backend') {
                    sh '''
                        echo "Checking JAR file..."
                        ls -lh target/*.jar
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('springboot-backend') {
                    sh '''
                        docker build \
                            -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Docker Login & Push Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-login',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$NEXUS_PASSWORD" | docker login ${NEXUS_REGISTRY} \
                            -u "$NEXUS_USERNAME" \
                            --password-stdin

                        docker tag \
                            ${IMAGE_NAME}:${IMAGE_TAG} \
                            ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push \
                            ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout ${NEXUS_REGISTRY}
                    '''
                }
            }
        }
    }
}