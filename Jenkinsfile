// ============================================================
// EMPLOYEE MANAGEMENT - FULL CI/CD PIPELINE
// PHASE 1 : BUILD JAR
// PHASE 2 : DOCKER BUILD + NEXUS PUSH
// PHASE 3 : HELM + ARGO CD + K3S
// ============================================================

pipeline {

    agent any

    tools {
        maven 'Maven3'  // This matches the name you configured
    }


    environment {

        // ====================================================
        // APPLICATION CONFIGURATION
        // ====================================================
        IMAGE_NAME = 'employee-management'
        IMAGE_TAG = "${BUILD_NUMBER}"

        // ====================================================
        // NEXUS REGISTRY
        // ====================================================
        NEXUS_REGISTRY = 'localhost:8082'

        // ====================================================
        // GITOPS REPOSITORY
        // ====================================================
        HELM_REPO_URL = 'https://github.com/sakthirangasamy/employee-management-gitops.git'
        HELM_BRANCH = 'main'

        // ====================================================
        // APPLICATION REPOSITORY
        // ====================================================
        APP_REPO_URL = 'https://github.com/sakthirangasamy/employee-management.git'
        APP_BRANCH = 'main'
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

                        java -version
                        mvn -version

                        echo "======================================"
                        echo "CLEANING AND BUILDING JAR"
                        echo "======================================"

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
                echo '''
                ==============================================
                PHASE 2 - DOCKER BUILD
                ==============================================
                '''

                script {
                    sh '''
                        echo "BUILD NUMBER : ${BUILD_NUMBER}"
                        echo "IMAGE        : ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"

                        docker --version

                        echo "======================================"
                        echo "BUILDING DOCKER IMAGE"
                        echo "======================================"

                        docker build \\
                            -t ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \\
                            -t ${NEXUS_REGISTRY}/${IMAGE_NAME}:latest \\
                            .

                        echo "======================================"
                        echo "DOCKER IMAGE BUILT SUCCESSFULLY"
                        echo "======================================"

                        docker images | grep ${IMAGE_NAME}
                    '''
                }
            }
        }

        stage('PHASE 2 - Verify Docker Image') {
            steps {
                sh '''
                    echo "======================================"
                    echo "VERIFY DOCKER IMAGE"
                    echo "======================================"

                    docker images | grep ${IMAGE_NAME}

                    echo ""
                    echo "Image Details:"
                    echo "  Name: ${NEXUS_REGISTRY}/${IMAGE_NAME}"
                    echo "  Tag:  ${IMAGE_TAG}"
                    echo "  Size: $(docker images ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} --format '{{.Size}}')"
                '''
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

                        echo "${NEXUS_PASSWORD}" | docker login ${NEXUS_REGISTRY} \\
                            -u "${NEXUS_USERNAME}" \\
                            --password-stdin

                        echo "======================================"
                        echo "PUSHING IMAGE TO NEXUS"
                        echo "======================================"

                        docker push ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${NEXUS_REGISTRY}/${IMAGE_NAME}:latest

                        echo "======================================"
                        echo "NEXUS PUSH SUCCESSFUL"
                        echo "======================================"

                        echo "Image pushed:"
                        echo "  ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                        echo "  ${NEXUS_REGISTRY}/${IMAGE_NAME}:latest"
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
                        echo "GITOPS REPOSITORY STRUCTURE"
                        echo "======================================"

                        echo "Repository: $(git remote get-url origin)"
                        echo "Branch:     $(git branch --show-current)"
                        echo "Commit:     $(git log -1 --oneline)"

                        echo ""
                        echo "Repository Files:"
                        find . -maxdepth 3 -type f | sort
                    '''
                }
            }
        }

        stage('PHASE 3 - Update Helm Image Tag') {
            steps {
                dir('helm-repo') {
                    sh '''
                        echo "======================================"
                        echo "CURRENT VALUES.YAML"
                        echo "======================================"

                        cat values.yaml

                        echo "======================================"
                        echo "UPDATING IMAGE TAG TO: ${IMAGE_TAG}"
                        echo "======================================"

                        # Update the image tag in values.yaml
                        sed -i "s/^  tag:.*/  tag: \\"${IMAGE_TAG}\\"/" values.yaml

                        echo "======================================"
                        echo "UPDATED VALUES.YAML"
                        echo "======================================"

                        cat values.yaml

                        echo "======================================"
                        echo "VALUES.YAML UPDATED SUCCESSFULLY"
                        echo "======================================"
                    '''
                }
            }
        }

        stage('PHASE 3 - Verify GitOps Changes') {
            steps {
                dir('helm-repo') {
                    sh '''
                        echo "======================================"
                        echo "VERIFYING GITOPS CHANGES"
                        echo "======================================"

                        echo "Git Status:"
                        git status

                        echo ""
                        echo "Values.yaml Diff:"
                        git diff -- values.yaml || echo "No differences found"
                    '''
                }
            }
        }

        stage('PHASE 3 - Commit Helm Changes') {
            steps {
                dir('helm-repo') {
                    sh '''
                        echo "======================================"
                        echo "CONFIGURING GIT"
                        echo "======================================"

                        git config user.name "sakthirangasamy"
                        git config user.email "sakthirangasamy2003@gmail.com"

                        echo "Git User:"
                        echo "  Name:  $(git config user.name)"
                        echo "  Email: $(git config user.email)"

                        echo "======================================"
                        echo "STAGING CHANGES"
                        echo "======================================"

                        git add values.yaml

                        echo "======================================"
                        echo "COMMITTING CHANGES"
                        echo "======================================"

                        git commit \\
                            -m "Update employee-management image to ${IMAGE_TAG}" \\
                            -m "Build Number: ${BUILD_NUMBER}" \\
                            -m "Image: ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}" \\
                            || echo "No changes to commit"
                    '''
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

                            git remote set-url origin \\
                            "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/sakthirangasamy/employee-management-gitops.git"

                            echo "Pushing to branch: ${HELM_BRANCH}"
                            git push origin ${HELM_BRANCH}

                            echo "======================================"
                            echo "GITOPS REPOSITORY UPDATED"
                            echo "======================================"

                            echo "Push successful!"
                            echo "Commit: $(git log -1 --oneline)"
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
                  Employee Management
                         Pod

                ┌─────────────────────────────────────────────┐
                │  Jenkins updates GitOps repository only.   │
                │  Argo CD handles the actual deployment.    │
                │  Auto-sync ensures continuous delivery.    │
                └─────────────────────────────────────────────┘

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

                ┌─────────────────────────────────────────────┐
                │  BUILD INFORMATION                         │
                ├─────────────────────────────────────────────┤
                │  Build Number       : ${BUILD_NUMBER}      │
                │  Docker Image       : ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} │
                │  Helm Image Tag     : ${IMAGE_TAG}         │
                └─────────────────────────────────────────────┘

                ┌─────────────────────────────────────────────┐
                │  GITOPS INFORMATION                        │
                ├─────────────────────────────────────────────┤
                │  Repository  : ${HELM_REPO_URL}            │
                │  Branch      : ${HELM_BRANCH}              │
                │  Updated     : values.yaml                 │
                └─────────────────────────────────────────────┘

                ┌─────────────────────────────────────────────┐
                │  DEPLOYMENT INFORMATION                     │
                ├─────────────────────────────────────────────┤
                │  Tool         : Argo CD                    │
                │  Kubernetes   : K3s                        │
                │  Sync Mode    : AUTO SYNC                  │
                └─────────────────────────────────────────────┘

                ==============================================

                Argo CD will detect the Git change
                and synchronize the application automatically.

                Monitor the deployment:
                1. Argo CD UI: http://localhost:8080
                2. K3s: kubectl get pods -n default
                3. Application: http://localhost:30080

                ==============================================
                """
            }
        }
    }


    // ========================================================
    // POST ACTIONS
    // ========================================================

    post {

        success {
            echo """
            ╔═════════════════════════════════════════════════════╗
            ║  ✅  FULL CI/CD PIPELINE COMPLETED SUCCESSFULLY   ║
            ╚═════════════════════════════════════════════════════╝

            ┌─────────────────────────────────────────────────────┐
            │  PIPELINE FLOW                                     │
            ├─────────────────────────────────────────────────────┤
            │                                                    │
            │  Phase 1  ──► JAR Created                          │
            │    │                                               │
            │    v                                               │
            │  Phase 2  ──► Docker Image Built                   │
            │    │                                               │
            │    v                                               │
            │  Nexus    ──► Image Pushed                        │
            │    │                                               │
            │    v                                               │
            │  Phase 3  ──► Helm Values Updated                  │
            │    │                                               │
            │    v                                               │
            │  GitHub   ──► GitOps Repository Updated           │
            │    │                                               │
            │    v                                               │
            │  Argo CD  ──► Auto Sync Triggered                 │
            │    │                                               │
            │    v                                               │
            │  K3s      ──► Application Deployed                │
            │    │                                               │
            │    v                                               │
            │  ✅  Employee Management Pod Running               │
            │                                                    │
            └─────────────────────────────────────────────────────┘

            ┌─────────────────────────────────────────────────────┐
            │  DEPLOYMENT DETAILS                                │
            ├─────────────────────────────────────────────────────┤
            │                                                    │
            │  Docker Image:                                     │
            │  ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}     │
            │                                                    │
            │  GitOps Commit:                                    │
            │  Updated values.yaml with tag ${IMAGE_TAG}         │
            │                                                    │
            │  Application:                                      │
            │  http://localhost:30080                           │
            │                                                    │
            └─────────────────────────────────────────────────────┘

            ╔═════════════════════════════════════════════════════╗
            ║  🎉  DEPLOYMENT SUCCESSFUL!                        ║
            ╚═════════════════════════════════════════════════════╝
            """
        }

        failure {
            echo """
            ╔═════════════════════════════════════════════════════╗
            ║  ❌  CI/CD PIPELINE FAILED                         ║
            ╚═════════════════════════════════════════════════════╝

            ┌─────────────────────────────────────────────────────┐
            │  TROUBLESHOOTING CHECKLIST                         │
            ├─────────────────────────────────────────────────────┤
            │                                                    │
            │  □  Phase 1: Maven Build                           │
            │     - Check pom.xml                                │
            │     - Verify Java/Maven versions                   │
            │     - Check for compilation errors                 │
            │                                                    │
            │  □  Phase 2: Docker Build                          │
            │     - Verify Dockerfile exists                     │
            │     - Check Docker daemon is running               │
            │     - Verify build context                         │
            │                                                    │
            │  □  Phase 2: Nexus Login                           │
            │     - Verify Nexus credentials                     │
            │     - Check Nexus is running                       │
            │     - Validate registry URL                        │
            │                                                    │
            │  □  Phase 2: Nexus Push                            │
            │     - Check disk space                             │
            │     - Verify Nexus repository                      │
            │     - Check network connectivity                   │
            │                                                    │
            │  □  Phase 3: GitHub Credentials                    │
            │     - Verify GitHub credentials ID                 │
            │     - Check token/permissions                      │
            │     - Validate repository access                   │
            │                                                    │
            │  □  Phase 3: GitOps Repository                     │
            │     - Verify repository exists                     │
            │     - Check branch name                            │
            │     - Validate values.yaml format                  │
            │                                                    │
            │  □  Phase 3: Git Commit/Push                       │
            │     - Check Git configuration                      │
            │     - Verify no merge conflicts                    │
            │     - Check commit message                         │
            │                                                    │
            │  □  Phase 3: Argo CD                               │
            │     - Verify Argo CD is running                    │
            │     - Check application sync status                │
            │     - Validate Helm chart                          │
            │                                                    │
            │  □  Phase 3: K3s/Kubernetes                        │
            │     - Check cluster health                         │
            │     - Verify resource limits                       │
            │     - Check pod status                             │
            │                                                    │
            └─────────────────────────────────────────────────────┘

            ╔═════════════════════════════════════════════════════╗
            ║  🔧  Review build logs for detailed error info    ║
            ╚═════════════════════════════════════════════════════╝
            """
        }

        aborted {
            echo """
            ⚠️  Pipeline aborted by user.
            """
        }

        cleanup {
            echo """
            ==============================================
            CLEANUP: Removing temporary files
            ==============================================
            """
            script {
                try {
                    sh '''
                        echo "Cleaning up helm-repo directory..."
                        rm -rf helm-repo || true
                    '''
                } catch (Exception e) {
                    echo "Cleanup warning: ${e.getMessage()}"
                }
            }
        }
    }
}