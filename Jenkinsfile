pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {

        // =========================
        // Application
        // =========================
        IMAGE_NAME = "employee-management"
        IMAGE_TAG  = "1.0"

        // =========================
        // Nexus Docker Registry
        // =========================
        NEXUS_URL = "172.25.93.123:8083"

        // =========================
        // Helm
        // =========================
        HELM_RELEASE = "employee-management"
        HELM_CHART   = "./helm/employee-management"

        // =========================
        // Kubernetes
        // =========================
        KUBECONFIG = "/var/jenkins_home/.kube/config"
    }

    stages {

        // ==========================================
        // 1. Checkout
        // ==========================================
        stage('Checkout') {

            steps {
                checkout scm
            }
        }


        // ==========================================
        // 2. Backend Build
        // ==========================================
        stage('Build Backend') {

            steps {

                dir('springboot-backend') {

                    sh '''
                        mvn clean package -DskipTests
                    '''
                }
            }
        }


        // ==========================================
        // 3. SonarQube Scan
        // ==========================================
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


        // ==========================================
        // 4. Quality Gate
        // ==========================================
        stage('Quality Gate') {

            steps {

                timeout(
                    time: 15,
                    unit: 'MINUTES'
                ) {

                    waitForQualityGate(
                        abortPipeline: true
                    )
                }
            }
        }


        // ==========================================
        // 5. Docker Build
        // ==========================================
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


        // ==========================================
        // 6. Docker Login Nexus
        // ==========================================
        stage('Docker Login Nexus') {

            steps {

                withCredentials([

                    usernamePassword(
                        credentialsId: 'nexus-login',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )

                ]) {

                    sh '''
                        echo "$NEXUS_PASSWORD" | \
                        docker login ${NEXUS_URL} \
                        -u "$NEXUS_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }


        // ==========================================
        // 7. Docker Tag
        // ==========================================
        stage('Docker Tag') {

            steps {

                sh '''
                    docker tag \
                    ${IMAGE_NAME}:${IMAGE_TAG} \
                    ${NEXUS_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }


        // ==========================================
        // 8. Docker Push Nexus
        // ==========================================
        stage('Docker Push Nexus') {

            steps {

                sh '''
                    docker push \
                    ${NEXUS_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }


        // ==========================================
        // 9. Helm Lint
        // ==========================================
        stage('Helm Lint') {

            steps {

                sh '''
                    helm lint ${HELM_CHART}
                '''
            }
        }


        // ==========================================
        // 10. Helm Template Validation
        // ==========================================
        stage('Helm Template') {

            steps {

                sh '''
                    helm template \
                    ${HELM_RELEASE} \
                    ${HELM_CHART} \
                    --set image.repository=${NEXUS_URL}/${IMAGE_NAME} \
                    --set image.tag=${IMAGE_TAG}
                '''
            }
        }


        // ==========================================
        // 11. Helm Deploy K3s
        // ==========================================
        stage('Helm Deploy K3s') {

            steps {

                sh '''
                    helm upgrade --install \
                    ${HELM_RELEASE} \
                    ${HELM_CHART} \
                    --namespace default \
                    --create-namespace \
                    --set image.repository=${NEXUS_URL}/${IMAGE_NAME} \
                    --set image.tag=${IMAGE_TAG}
                '''
            }
        }


        // ==========================================
        // 12. Kubernetes Rollout
        // ==========================================
        stage('Kubernetes Rollout') {

            steps {

                sh '''
                    kubectl rollout status \
                    deployment/${HELM_RELEASE} \
                    --timeout=180s
                '''
            }
        }


        // ==========================================
        // 13. Kubernetes Status
        // ==========================================
        stage('Kubernetes Status') {

            steps {

                sh '''
                    echo "================ PODS ================"

                    kubectl get pods -o wide

                    echo "================ SERVICES ================"

                    kubectl get services

                    echo "================ DEPLOYMENTS ================"

                    kubectl get deployments

                    echo "================ HELM ================"

                    helm list
                '''
            }
        }
    }


    // ==========================================
    // POST ACTIONS
    // ==========================================
    post {

        success {

            echo '''
            ==========================================
            CI/CD PIPELINE SUCCESS
            ==========================================
            Docker image pushed to Nexus.
            Helm deployment completed.
            Kubernetes rollout completed.
            ==========================================
            '''
        }

        failure {

            echo '''
            ==========================================
            CI/CD PIPELINE FAILED
            ==========================================
            Check the failed stage and Jenkins console.
            ==========================================
            '''
        }

        always {

            sh '''
                docker logout ${NEXUS_URL} || true
            '''
        }
    }
}