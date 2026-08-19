// ============================================================
// PHASE - 3: HELM + ARGO CD DEPLOYMENT
// ============================================================

pipeline {
    agent any

    environment {
        IMAGE_NAME = 'employee-management'
        IMAGE_TAG = "${BUILD_NUMBER}"
        NEXUS_REGISTRY = 'localhost:8082'
        HELM_REPO_URL = 'https://github.com/YOUR-ORG/employee-management-helm.git'
        HELM_BRANCH = 'main'
    }

    stages {
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
                        sed -i "s/^  tag:.*/  tag: \\"${IMAGE_TAG}\\"/" values.yaml

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
                        git commit -m "Update employee-management image to ${IMAGE_TAG}" || echo "No changes to commit"
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
                            echo "✅ HELM REPOSITORY UPDATED"
                            echo "======================================"
                        '''
                    }
                }
            }
        }

        stage('Argo CD Deployment') {
            steps {
                echo '''
                ==============================================
                🚀 ARGO CD DEPLOYMENT
                ==============================================

                Helm Git repository updated.

                Argo CD will detect the Git change automatically.

                    GitHub
                       ↓
                    Argo CD (Auto Sync)
                       ↓
                    Helm Chart
                       ↓
                    K3s Cluster
                       ↓
              Employee Management Pod

                ==============================================
                '''
            }
        }

        stage('Deployment Verification') {
            steps {
                echo """
                ==============================================
                ✅ DEPLOYMENT INFORMATION
                ==============================================

                Build Number: ${BUILD_NUMBER}
                Docker Image: ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                Helm Image Tag: ${IMAGE_TAG}
                Argo CD: AUTO SYNC ENABLED
                Kubernetes: K3s

                ==============================================
                🔍 Check Deployment:
                kubectl get pods -n <namespace>
                kubectl get svc -n <namespace>
                ==============================================
                """
            }
        }
    }

    post {
        success {
            echo """
            ==============================================
            ✅ PHASE 3 COMPLETED SUCCESSFULLY
            ==============================================

            🎉 Full CI/CD Pipeline Completed!

            ✅ JAR Created (Phase 1)
            ✅ Docker Image Built & Pushed (Phase 2)
            ✅ Helm Updated & Deployed (Phase 3)

            Argo CD will synchronize automatically to K3s.

            ==============================================
            """
        }
        failure {
            echo """
            ==============================================
            ❌ PHASE 3 FAILED
            ==============================================
            Check Helm chart and GitHub credentials.
            ==============================================
            """
        }
    }
}