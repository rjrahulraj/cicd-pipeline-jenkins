pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['main', 'dev'],
            description: 'Select the environment to deploy'
        )
    }

    tools {
        nodejs 'Node 7.8.0'
    }

    environment {
        REPO_URL = 'https://github.com/rjrahulraj/cicd-pipeline-jenkins.git'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${params.ENVIRONMENT}",
                    url: "${REPO_URL}"
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Installing dependencies..."
                    npm install

                    echo "Building application..."
                    npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    npm test -- --watchAll=false
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'main') {
                        env.IMAGE_NAME = 'nodemain:v1.0'
                    } else {
                        env.IMAGE_NAME = 'nodedev:v1.0'
                    }

                    sh """
                        echo "Building Docker image: ${IMAGE_NAME}"
                        docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'main') {
                        env.CONTAINER_NAME = 'nodemain'
                        env.HOST_PORT = '3000'
                    } else {
                        env.CONTAINER_NAME = 'nodedev'
                        env.HOST_PORT = '3001'
                    }

                    sh """
                        echo "Deploying ${CONTAINER_NAME}"
                        echo "Host port: ${HOST_PORT}"

                        docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p ${HOST_PORT}:3000 \
                            -e HOST=0.0.0.0 \
                            -e PORT=3000 \
                            ${IMAGE_NAME}

                        echo "Deployment completed."

                        docker ps
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Manual deployment completed successfully for ${params.ENVIRONMENT}"
        }

        failure {
            echo "Manual deployment failed for ${params.ENVIRONMENT}"
        }
    }
}