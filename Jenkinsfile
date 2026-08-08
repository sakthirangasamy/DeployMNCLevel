pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {

        // Nexus
        NEXUS_URL = '172.25.93.123:8083'
        IMAGE_NAME = 'employee-management'
        IMAGE_TAG = '1.0'

        // Full Docker image
        DOCKER_IMAGE = "${NEXUS_URL}/${IMAGE_NAME}:${IMAGE_TAG}"

        // Helm
        HELM_RELEASE = 'employee-management'
        HELM_CHART = './helm/employee-management'
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
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('springboot-backend') {
                    sh """
                        docker build \
                        -t ${DOCKER_IMAGE} \
                        .
                    """
                }
            }
        }

        stage('Docker Login Nexus') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-login',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {

                    sh """
                        echo "\$NEXUS_PASSWORD" | \
                        docker login ${NEXUS_URL} \
                        -u "\$NEXUS_USERNAME" \
                        --password-stdin
                    """
                }
            }
        }

        stage('Docker Push Nexus') {
            steps {

                sh """
                    docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Helm Lint') {
            steps {

                sh """
                    helm lint ${HELM_CHART}
                """
            }
        }

        stage('Helm Deploy K3s') {
            steps {

                sh """
                    helm upgrade --install \
                    ${HELM_RELEASE} \
                    ${HELM_CHART} \
                    --set image.repository=${NEXUS_URL}/${IMAGE_NAME} \
                    --set image.tag=${IMAGE_TAG}
                """
            }
        }

        stage('Kubernetes Status') {
            steps {

                sh """
                    kubectl get pods
                    kubectl get services
                """
            }
        }
    }

    post {

        success {
            echo 'Deployment successful!'
        }

        failure {
            echo 'Deployment failed!'
        }

        always {
            sh 'docker logout ${NEXUS_URL} || true'
        }
    }
}