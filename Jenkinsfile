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

        stage('Maven Build') {
            steps {
                dir('springboot-backend') {
                    sh '''
                        echo "Building Spring Boot application..."

                        mvn clean package -DskipTests

                        echo "Build completed."
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

                        echo "JAR file generated successfully."
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('springboot-backend') {
                    sh '''
                        echo "Building Docker image..."

                        docker build \
                            -t ${IMAGE_NAME}:${IMAGE_TAG} .

                        echo "Docker image created:"
                        docker images | grep ${IMAGE_NAME}
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
                        echo "Logging into Nexus..."

                        echo "$NEXUS_PASSWORD" | \
                        docker login ${NEXUS_REGISTRY} \
                        -u "$NEXUS_USERNAME" \
                        --password-stdin

                        echo "Tagging Docker image..."

                        docker tag \
                            ${IMAGE_NAME}:${IMAGE_TAG} \
                            ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        echo "Pushing image to Nexus..."

                        docker push \
                            ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout ${NEXUS_REGISTRY}

                        echo "Docker image pushed successfully."
                    '''
                }
            }
        }

        stage('Verify Docker Image') {
            steps {
                sh '''
                    echo "Verifying Docker image..."

                    docker images | grep ${IMAGE_NAME}
                '''
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "Phase 2 completed successfully!"
            echo "JAR built successfully."
            echo "Docker image built successfully."
            echo "Docker image pushed to Nexus."
            echo "Image: ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "Phase 2 failed!"
            echo "Check Jenkins console output."
            echo "======================================"
        }
    }
}