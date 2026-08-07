pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }


    environment {

        IMAGE_NAME = "employee-management"
        IMAGE_TAG = "1.0"

        // Nexus Docker Registry
        NEXUS_URL = "localhost:8083"

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

                    sh '''

                    docker build \
                    -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    '''

                }
            }
        }





        stage('Docker Push Nexus') {

            steps {


                withCredentials([

                    usernamePassword(
                        credentialsId: 'nexus-login',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )

                ]) {


                    sh '''

                    echo $NEXUS_PASSWORD | docker login ${NEXUS_URL} \
                    -u $NEXUS_USERNAME \
                    --password-stdin



                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                    ${NEXUS_URL}/${IMAGE_NAME}:${IMAGE_TAG}



                    docker push \
                    ${NEXUS_URL}/${IMAGE_NAME}:${IMAGE_TAG}



                    docker logout ${NEXUS_URL}

                    '''

                }

            }
        }

    }



    post {


        success {

            echo 'CI Pipeline completed successfully.'

        }


        failure {

            echo 'CI Pipeline failed.'

        }


    }

}