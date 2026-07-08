pipeline {
    agent any

    tools {
        nodejs 'Node18'
    }

    environment {
        IMAGE_NAME = "cicd-pipeline"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test -- --watchAll=false'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${env.BRANCH_NAME}")
                }
            }
        }

        stage('Deploy') {
            steps {
                script {

                    if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.APP_PORT = '3001'
                    }

                    sh """
                    docker rm -f ${IMAGE_NAME}-${env.BRANCH_NAME} || true

                    docker run -d \
                      --name ${IMAGE_NAME}-${env.BRANCH_NAME} \
                      -p ${env.APP_PORT}:3000 \
                      ${IMAGE_NAME}:${env.BRANCH_NAME}
                    """
                }
            }
        }
    }
}
