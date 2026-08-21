// ============================================================
// EMPLOYEE MANAGEMENT - FULL CI/CD PIPELINE
// ============================================================

pipeline {

    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        IMAGE_NAME = 'employee-management'
        IMAGE_TAG = "${BUILD_NUMBER}"
        
        NEXUS_REGISTRY = 'localhost:8083'
        NEXUS_REPOSITORY = 'employee-docker'
        
        HELM_REPO_URL = 'https://github.com/sakthirangasamy/employee-management-gitops.git'
        HELM_BRANCH = 'main'
    }

    stages {

        // ====================================================
        // PHASE 1: BUILD JAR
        // ====================================================

        stage('PHASE 1 - Checkout Application') {
            steps {
                echo '''
                ==============================================
                PHASE 1 - CHECKOUT APPLICATION
                ==============================================
                '''
                checkout scm
            }
        }

        stage('PHASE 1 - Build JAR') {
            steps {
                dir('springboot-backend') {
                    sh '''
                        echo "======================================"
                        echo "PHASE 1 - MAVEN BUILD"
                        echo "======================================"
                        
                        mvn --version
                        mvn clean package -DskipTests
                        
                        echo "======================================"
                        echo "JAR CREATED SUCCESSFULLY"
                        echo "======================================"
                        
                        ls -lh target/*.jar
                    '''
                }
            }
        }

        // ====================================================
        // PHASE 2: DOCKER BUILD + NEXUS PUSH
        // ====================================================

        stage('PHASE 2 - Docker Build') {
            steps {
                dir('springboot-backend') {
                    sh """
                        echo "======================================"
                        echo "PHASE 2 - DOCKER BUILD"
                        echo "======================================"
                        
                        echo "Building image: ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                        
                        docker build \\
                            -t ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \\
                            -t ${NEXUS_REGISTRY}/${IMAGE_NAME}:latest \\
                            .
                        
                        echo "======================================"
                        echo "DOCKER IMAGE BUILT SUCCESSFULLY"
                        echo "======================================"
                        
                        docker images | grep ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('PHASE 2 - Verify Docker Image') {
            steps {
                sh """
                    echo "======================================"
                    echo "VERIFY DOCKER IMAGE"
                    echo "======================================"
                    
                    docker images | grep ${IMAGE_NAME}
                    
                    echo ""
                    echo "Image Details:"
                    echo "  Registry: ${NEXUS_REGISTRY}"
                    echo "  Repository: ${NEXUS_REPOSITORY}"
                    echo "  Image: ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    echo "  Size: \$(docker images ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} --format '{{.Size}}')"
                """
            }
        }

        stage('PHASE 2 - Push Image to Nexus') {
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
                        echo "LOGIN TO NEXUS REGISTRY"
                        echo "======================================"

                        echo "Nexus URL: $NEXUS_REGISTRY"

                        echo "$NEXUS_PASSWORD" | docker login "$NEXUS_REGISTRY" -u "$NEXUS_USERNAME" --password-stdin

                        echo "======================================"
                        echo "PUSHING IMAGE TO NEXUS"
                        echo "======================================"

                        docker push "$NEXUS_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
                        docker push "$NEXUS_REGISTRY/$IMAGE_NAME:latest"

                        echo "======================================"
                        echo "NEXUS PUSH SUCCESSFUL"
                        echo "======================================"

                        echo "✅ Images pushed to Nexus:"
                        echo "   $NEXUS_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
                        echo "   $NEXUS_REGISTRY/$IMAGE_NAME:latest"
                    '''
                }
            }
        }

        // ====================================================
        // PHASE 3: GITOPS + ARGO CD DEPLOYMENT
        // ====================================================

        stage('PHASE 3 - Checkout Helm Repository') {
            steps {
                echo '''
                ==============================================
                PHASE 3 - CHECKOUT GITOPS REPOSITORY
                ==============================================
                '''
                dir('helm-repo') {
                    git(
                        branch: "${HELM_BRANCH}",
                        url: "${HELM_REPO_URL}"
                    )
                    
                    sh '''
                        echo "======================================"
                        echo "GITOPS REPOSITORY"
                        echo "======================================"
                        
                        echo "Repository: $(git remote get-url origin)"
                        echo "Branch:     $(git branch --show-current)"
                        ls -la
                    '''
                }
            }
        }

        stage('PHASE 3 - Update Helm Image Tag') {
            steps {
                dir('helm-repo') {
                    sh """
                        echo "======================================"
                        echo "UPDATING IMAGE TAG TO: ${IMAGE_TAG}"
                        echo "======================================"
                        
                        echo "Current values.yaml:"
                        cat values.yaml
                        
                        sed -i "s/^  tag:.*/  tag: \\"${IMAGE_TAG}\\"/" values.yaml
                        
                        echo "======================================"
                        echo "UPDATED VALUES.YAML"
                        echo "======================================"
                        
                        cat values.yaml
                    """
                }
            }
        }

        stage('PHASE 3 - Verify GitOps Changes') {
            steps {
                dir('helm-repo') {
                    sh '''
                        echo "======================================"
                        echo "VERIFY GITOPS CHANGES"
                        echo "======================================"
                        
                        git status
                        echo ""
                        echo "Changes in values.yaml:"
                        git diff -- values.yaml
                    '''
                }
            }
        }

        stage('PHASE 3 - Commit Helm Changes') {
            steps {
                dir('helm-repo') {
                    sh """
                        echo "======================================"
                        echo "COMMITTING CHANGES"
                        echo "======================================"
                        
                        git config user.name "sakthirangasamy"
                        git config user.email "sakthirangasamy2003@gmail.com"
                        
                        git add values.yaml
                        git commit -m "Update employee-management image to ${IMAGE_TAG}" || echo "No changes to commit"
                    """
                }
            }
        }

        stage('PHASE 3 - Push Helm Changes') {
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
                            echo "PUSHING TO GITHUB"
                            echo "======================================"

                            git remote set-url origin "https://$GIT_USERNAME:$GIT_PASSWORD@github.com/sakthirangasamy/employee-management-gitops.git"

                            git push origin "$HELM_BRANCH"

                            echo "======================================"
                            echo "GITOPS REPOSITORY UPDATED"
                            echo "======================================"
                        '''
                    }
                }
            }
        }

        stage('PHASE 3 - Argo CD Deployment') {
            steps {
                echo '''
                ==============================================
                ARGO CD GITOPS DEPLOYMENT
                ==============================================

                         GitHub
                            |
                            v
                         Argo CD
                            |
                       Auto Sync
                            |
                            v
                        Helm Chart
                            |
                            v
                           K3s
                            |
                            v
                  Employee Management Pod

                Jenkins updates GitOps repository only.
                Argo CD handles the actual deployment.
                Auto-sync ensures continuous delivery.

                ==============================================
                '''
            }
        }

        stage('PHASE 3 - Deployment Verification') {
            steps {
                echo """
                ==============================================
                DEPLOYMENT INFORMATION
                ==============================================

                Build Number       : ${BUILD_NUMBER}
                Docker Image       : ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                Image Tag          : ${IMAGE_TAG}

                Nexus Registry     : ${NEXUS_REGISTRY}
                Nexus Repository   : ${NEXUS_REPOSITORY}
                Nexus UI           : http://localhost:8083

                GitOps Repository  : ${HELM_REPO_URL}
                Git Branch         : ${HELM_BRANCH}

                Deployment Tool    : Argo CD
                Kubernetes         : K3s
                Sync Mode          : AUTO SYNC

                ==============================================

                Argo CD will detect the Git change
                and synchronize the application automatically.

                Monitor the deployment:
                1. Argo CD UI: http://localhost:8080
                2. K3s: kubectl get pods -n default
                3. Application: http://localhost:30080
                4. Nexus: http://localhost:8083/#browse/browse:${NEXUS_REPOSITORY}

                ==============================================
                """
            }
        }
    }

    post {
        success {
            echo """
            ==============================================
            ✅  CI/CD PIPELINE COMPLETED SUCCESSFULLY
            ==============================================

            DEPLOYMENT SUMMARY
            -------------------
            ✅ Phase 1: JAR Built
            ✅ Phase 2: Docker Image Built
            ✅ Phase 2: Image Pushed to Nexus
            ✅ Phase 3: GitOps Repository Updated
            ✅ Phase 3: Argo CD Sync Triggered

            IMAGE DETAILS
            --------------
            Image: ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
            Registry: localhost:8083
            Repository: ${NEXUS_REPOSITORY}

            View in Nexus:
            http://localhost:8083/#browse/browse:${NEXUS_REPOSITORY}

            ==============================================
            🎉  DEPLOYMENT SUCCESSFUL!
            ==============================================
            """
        }

        failure {
            echo '''
            ==============================================
            ❌  CI/CD PIPELINE FAILED
            ==============================================

            TROUBLESHOOTING CHECKLIST
            -------------------------
            1. Nexus is running on port 8083
            2. Nexus credentials are correct
            3. Docker daemon is running
            4. GitHub credentials are correct
            5. GitOps repository exists
            6. values.yaml exists in GitOps repo
            7. Argo CD is running
            8. K3s cluster is healthy

            Check Nexus: http://localhost:8083
            Check Jenkins logs for more details.

            ==============================================
            '''
        }

        always {
            echo """
            ==============================================
            BUILD COMPLETED AT: ${new Date()}
            BUILD NUMBER: ${BUILD_NUMBER}
            ==============================================
            """
        }
    }
}