// ============================================================
// PHASE - 1 + PHASE - 2 + PHASE - 3
// Jenkins CI/CD Pipeline
//
// Flow:
//
// GitHub Application
//       ↓
// Build & Test
//       ↓
// SonarQube
//       ↓
// Quality Gate
//       ↓
// JAR
//       ↓
// Docker Build
//       ↓
// Nexus Push
//       ↓
// Checkout Helm Repository
//       ↓
// Update values.yaml image tag
//       ↓
// Git Commit & Push
//       ↓
// Argo CD Auto Sync
//       ↓
// K3s Deployment
// ============================================================


pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {

        // ========================================================
        // Docker
        // ========================================================

        IMAGE_NAME = 'employee-management'
        IMAGE_TAG = "${BUILD_NUMBER}"

        // Nexus Docker Registry
        NEXUS_REGISTRY = 'localhost:8082'


        // ========================================================
        // Helm Git Repository
        // ========================================================

        HELM_REPO_URL = 'https://github.com/YOUR-ORG/employee-management-helm.git'
        HELM_BRANCH = 'main'


        // ========================================================
        // SonarQube
        // ========================================================

        SONAR_PROJECT_KEY = 'employee-management'
        SONAR_PROJECT_NAME = 'employee-management'
    }


    stages {


        // ========================================================
        // PHASE - 1
        // ========================================================


        stage('Checkout') {

            steps {

                echo '======================================'
                echo 'CHECKOUT APPLICATION SOURCE CODE'
                echo '======================================'

                checkout scm

                sh '''
                    echo "Current workspace:"
                    pwd

                    echo "Application files:"
                    ls -la
                '''
            }
        }


        stage('Build & Test') {

            steps {

                dir('springboot-backend') {

                    sh '''

                        echo "======================================"
                        echo "JAVA VERSION"
                        echo "======================================"

                        java -version


                        echo "======================================"
                        echo "MAVEN VERSION"
                        echo "======================================"

                        mvn -version


                        echo "======================================"
                        echo "BUILD & TEST"
                        echo "======================================"

                        mvn clean verify -DforkCount=0

                    '''
                }
            }
        }


        stage('SonarQube Scan') {

            steps {

                dir('springboot-backend') {

                    withSonarQubeEnv('SonarQube') {

                        sh '''

                            echo "======================================"
                            echo "SONARQUBE SCAN"
                            echo "======================================"

                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.4.0.6343:sonar \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.projectName=${SONAR_PROJECT_NAME}

                        '''
                    }
                }
            }
        }


        stage('Quality Gate') {

            steps {

                echo '======================================'
                echo 'WAITING FOR SONARQUBE QUALITY GATE'
                echo '======================================'

                timeout(time: 10, unit: 'MINUTES') {

                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage('Package JAR') {

            steps {

                dir('springboot-backend') {

                    sh '''

                        echo "======================================"
                        echo "PACKAGE JAR"
                        echo "======================================"

                        mvn package -DskipTests

                        echo "Generated JAR:"
                        ls -lh target/*.jar

                    '''
                }
            }
        }


        // ========================================================
        // PHASE - 2
        // ========================================================


        stage('Build Spring Boot') {

            steps {

                dir('springboot-backend') {

                    sh '''

                        echo "======================================"
                        echo "BUILD SPRING BOOT"
                        echo "======================================"

                        mvn clean package -DskipTests

                    '''
                }
            }
        }


        stage('Verify JAR') {

            steps {

                dir('springboot-backend') {

                    sh '''

                        echo "======================================"
                        echo "VERIFY JAR"
                        echo "======================================"

                        ls -lh target/*.jar

                    '''
                }
            }
        }


        stage('Docker Build') {

            steps {

                dir('springboot-backend') {

                    sh '''

                        echo "======================================"
                        echo "DOCKER BUILD"
                        echo "======================================"

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

                        echo "======================================"
                        echo "NEXUS LOGIN"
                        echo "======================================"

                        echo "$NEXUS_PASSWORD" | docker login ${NEXUS_REGISTRY} \
                            -u "$NEXUS_USERNAME" \
                            --password-stdin


                        echo "======================================"
                        echo "DOCKER TAG"
                        echo "======================================"

                        docker tag \
                            ${IMAGE_NAME}:${IMAGE_TAG} \
                            ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}


                        echo "======================================"
                        echo "PUSH IMAGE TO NEXUS"
                        echo "======================================"

                        docker push \
                            ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}


                        echo "======================================"
                        echo "IMAGE PUSHED"
                        echo "======================================"

                        echo "${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"


                        docker logout ${NEXUS_REGISTRY}

                    '''
                }
            }
        }


        // ========================================================
        // PHASE - 3
        // HELM + ARGO CD
        // ========================================================


        stage('Checkout Helm Repository') {

            steps {

                echo '======================================'
                echo 'CHECKOUT HELM REPOSITORY'
                echo '======================================'


                dir('helm-repo') {

                    git(

                        branch: "${HELM_BRANCH}",

                        url: "${HELM_REPO_URL}"

                    )
                }


                sh '''

                    echo "Helm repository structure:"

                    find helm-repo -maxdepth 3 -type f

                '''
            }
        }


        stage('Update Helm Image Tag') {

            steps {

                dir('helm-repo') {

                    sh '''

                        echo "======================================"
                        echo "OLD VALUES.YAML"
                        echo "======================================"

                        cat values.yaml


                        echo "======================================"
                        echo "UPDATING IMAGE TAG"
                        echo "======================================"

                        sed -i \
                        "s/^  tag:.*/  tag: \\"${IMAGE_TAG}\\"/" \
                        values.yaml


                        echo "======================================"
                        echo "NEW VALUES.YAML"
                        echo "======================================"

                        cat values.yaml

                    '''
                }
            }
        }


        stage('Verify Helm Chart') {

            steps {

                dir('helm-repo') {

                    sh '''

                        echo "======================================"
                        echo "VERIFY HELM CHART"
                        echo "======================================"

                        helm version


                        echo "Helm lint:"

                        helm lint .

                    '''
                }
            }
        }


        stage('Commit Helm Changes') {

            steps {

                dir('helm-repo') {

                    sh '''

                        echo "======================================"
                        echo "GIT CONFIG"
                        echo "======================================"

                        git config user.name "Jenkins"

                        git config user.email "jenkins@localhost"


                        echo "======================================"
                        echo "GIT STATUS"
                        echo "======================================"

                        git status


                        echo "======================================"
                        echo "GIT ADD"
                        echo "======================================"

                        git add values.yaml


                        echo "======================================"
                        echo "GIT COMMIT"
                        echo "======================================"

                        git commit \
                            -m "Update employee-management image to ${IMAGE_TAG}" \
                            || echo "No changes to commit"

                    '''
                }
            }
        }


        stage('Push Helm Changes') {

            steps {

                dir('helm-repo') {

                    withCredentials([

                        usernamePassword(

                            credentialsId: 'github-credentials',

                            usernameVariable: 'GIT_USERNAME',

                            passwordVariable: 'GIT_PASSWORD'

                        )

                    ]) {

                        sh '''

                            echo "======================================"
                            echo "PUSH HELM CHANGES TO GITHUB"
                            echo "======================================"


                            git remote set-url origin \
                            https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/YOUR-ORG/employee-management-helm.git


                            git push origin ${HELM_BRANCH}


                            echo "======================================"
                            echo "HELM REPOSITORY UPDATED"
                            echo "======================================"

                        '''
                    }
                }
            }
        }


        // ========================================================
        // ARGO CD
        // ========================================================


        stage('Argo CD Deployment Triggered') {

            steps {

                echo '''

                ==============================================
                ARGO CD DEPLOYMENT
                ==============================================

                Helm Git repository updated.

                Argo CD will detect the Git change.

                    GitHub
                       ↓
                    Argo CD
                       ↓
                    Helm
                       ↓
                    K3s
                       ↓
              Employee Management Pod

                ==============================================

                '''

            }
        }


        // ========================================================
        // FINAL VERIFICATION
        // ========================================================


        stage('Deployment Information') {

            steps {

                echo """

                ==============================================
                CI/CD PIPELINE COMPLETED
                ==============================================

                Build Number:

                ${BUILD_NUMBER}


                Docker Image:

                ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}


                Helm Image Tag:

                ${IMAGE_TAG}


                Argo CD:

                AUTO SYNC ENABLED


                Kubernetes:

                K3s


                ==============================================

                """

            }
        }
    }


    // ============================================================
    // POST
    // ============================================================


    post {

        success {

            echo """

            ==============================================
            SUCCESS
            ==============================================

            Build #${BUILD_NUMBER} completed successfully.

            Docker image pushed to Nexus.

            Helm values.yaml updated.

            Helm repository pushed to GitHub.

            Argo CD will synchronize automatically.

            ==============================================

            """

        }


        failure {

            echo """

            ==============================================
            PIPELINE FAILED
            ==============================================

            Build #${BUILD_NUMBER} failed.

            Please check Jenkins Console Output.

            ==============================================

            """

        }
    }
}