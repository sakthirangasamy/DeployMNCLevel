// ============================================================
// PHASE - 3: GITOPS + ARGO CD DEPLOYMENT
// ============================================================

pipeline {

    agent any

    environment {
        IMAGE_NAME = 'employee-management'
        IMAGE_TAG = "${BUILD_NUMBER}"

        NEXUS_REGISTRY = 'localhost:8082'

        HELM_REPO_URL = 'https://github.com/sakthirangasamy/employee-management-gitops.git'
        HELM_BRANCH = 'main'
    }

    stages {

        // ========================================================
        // 1. CHECKOUT GITOPS / HELM REPOSITORY
        // ========================================================
        stage('Checkout Helm Repository') {
            steps {

                echo '======================================'
                echo 'CHECKOUT GITOPS / HELM REPOSITORY'
                echo '======================================'

                dir('helm-repo') {

                    git(
                        branch: "${HELM_BRANCH}",
                        url: "${HELM_REPO_URL}"
                    )

                    sh '''
                        echo "======================================"
                        echo "GITOPS REPOSITORY STRUCTURE"
                        echo "======================================"

                        find . -maxdepth 3 -type f | sort
                    '''
                }
            }
        }


        // ========================================================
        // 2. UPDATE HELM IMAGE TAG
        // ========================================================
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

                        sed -i "s/^  tag:.*/  tag: \\"${IMAGE_TAG}\\"/" values.yaml

                        echo "======================================"
                        echo "NEW VALUES.YAML"
                        echo "======================================"

                        cat values.yaml
                    '''
                }
            }
        }


        // ========================================================
        // 3. VERIFY GIT CHANGES
        // ========================================================
        stage('Verify GitOps Changes') {
            steps {

                dir('helm-repo') {

                    sh '''
                        echo "======================================"
                        echo "VERIFY GITOPS CHANGES"
                        echo "======================================"

                        git status

                        echo "======================================"
                        echo "VALUES.YAML DIFF"
                        echo "======================================"

                        git diff -- values.yaml
                    '''
                }
            }
        }


        // ========================================================
        // 4. COMMIT HELM CHANGES
        // ========================================================
        stage('Commit Helm Changes') {
            steps {

                dir('helm-repo') {

                    sh '''
                        echo "======================================"
                        echo "GIT CONFIGURATION"
                        echo "======================================"

                        git config user.name "sakthirangasamy"
                        git config user.email "sakthirangasamy2003@gmail.com"

                        echo "======================================"
                        echo "GIT USER"
                        echo "======================================"

                        git config user.name
                        git config user.email

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


        // ========================================================
        // 5. PUSH GITOPS CHANGES TO GITHUB
        // ========================================================
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
                            echo "PUSH GITOPS CHANGES TO GITHUB"
                            echo "======================================"

                            git remote set-url origin \
                            "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/sakthirangasamy/employee-management-gitops.git"

                            git push origin ${HELM_BRANCH}

                            echo "======================================"
                            echo "GITOPS REPOSITORY UPDATED"
                            echo "======================================"
                        '''
                    }
                }
            }
        }


        // ========================================================
        // 6. ARGO CD DEPLOYMENT
        // ========================================================
        stage('Argo CD Deployment') {
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

                Jenkins only updates GitOps repository.

                Argo CD is responsible for deployment.

                ==============================================
                '''
            }
        }


        // ========================================================
        // 7. DEPLOYMENT INFORMATION
        // ========================================================
        stage('Deployment Verification') {
            steps {

                echo """
                ==============================================
                DEPLOYMENT INFORMATION
                ==============================================

                Build Number       : ${BUILD_NUMBER}
                Docker Image       : ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                Helm Image Tag     : ${IMAGE_TAG}

                GitOps Repository  :
                ${HELM_REPO_URL}

                Git Branch         : ${HELM_BRANCH}

                Deployment Tool    : Argo CD
                Kubernetes         : K3s
                Sync Mode          : AUTO SYNC

                ==============================================

                Argo CD will detect the Git change
                and synchronize the application automatically.

                ==============================================
                """
            }
        }
    }


    // ============================================================
    // POST ACTIONS
    // ============================================================
    post {

        success {

            echo '''
            ==============================================
            PHASE 3 COMPLETED SUCCESSFULLY
            ==============================================

            Full CI/CD Pipeline Completed!

            Phase 1
                |
                v
            JAR Created
                |
                v
            Phase 2
                |
                v
            Docker Image Built & Pushed
                |
                v
            Nexus
                |
                v
            Phase 3
                |
                v
            GitOps Helm Values Updated
                |
                v
            GitHub
                |
                v
            Argo CD
                |
                v
            K3s
                |
                v
            Employee Management Pod

            ==============================================
            '''
        }

        failure {

            echo '''
            ==============================================
            PHASE 3 FAILED
            ==============================================

            Check:

            1. GitHub credentials
            2. GitOps repository
            3. values.yaml
            4. Git commit
            5. Git push
            6. Argo CD Auto Sync

            ==============================================
            '''
        }
    }
}