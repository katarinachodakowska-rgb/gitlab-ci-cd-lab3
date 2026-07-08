pipeline {

    agent any

    tools {
        nodejs 'Node18'
    }

    environment {
        IMAGE_NAME = "cicd-react-app"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }


        stage('Build') {
            steps {
                sh '''
                export NODE_OPTIONS=--dns-result-order=ipv4first

                npm config set registry https://registry.npmjs.org/
                npm config set fetch-retries 5
                npm config set fetch-retry-mintimeout 20000
                npm config set fetch-retry-maxtimeout 120000

                npm install --prefer-online

                npm run build
                '''
            }
        }


        stage('Test') {
            steps {
                sh '''
                npm test -- --watchAll=false
                '''
            }
        }


        stage('Build Docker Image') {
            steps {

                script {

                    if (env.BRANCH_NAME == 'main') {
                        env.PORT = "3000"
                    }
                    else if (env.BRANCH_NAME == 'dev') {
                        env.PORT = "3001"
                    }
                    else {
                        env.PORT = "3000"
                    }


                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Application port: ${env.PORT}"


                    sh """
                    docker build \
                    -t ${IMAGE_NAME}:${env.BRANCH_NAME} .
                    """
                }
            }
        }


        stage('Deploy') {
            steps {

                script {

                    sh """

                    docker stop ${IMAGE_NAME}-${env.BRANCH_NAME} || true

                    docker rm ${IMAGE_NAME}-${env.BRANCH_NAME} || true


                    docker run -d \
                    --name ${IMAGE_NAME}-${env.BRANCH_NAME} \
                    -p ${env.PORT}:3000 \
                    ${IMAGE_NAME}:${env.BRANCH_NAME}

                    """
                }
            }
        }
    }


    post {

        success {
            echo "Deployment successful!"
            echo "Running on port ${PORT}"
        }


        failure {
            echo "Pipeline failed"
        }
    }
}
