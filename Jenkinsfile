pipeline {
    agent any

    tools {
        nodejs 'Node 7.8.0'
    }

    environment {
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        HOST_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
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
                    echo "Building application..."
                    npm install
                    npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    ./scripts/test.sh
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image: ${IMAGE_NAME}"
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying ${CONTAINER_NAME} on port ${HOST_PORT}"

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:3000 \
                        -e HOST=0.0.0.0 \
                        -e PORT=3000 \
                        ${IMAGE_NAME}

                    echo "Deployment completed."
                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully for ${env.BRANCH_NAME}"
        }

        failure {
            echo "Pipeline failed for ${env.BRANCH_NAME}"
        }
    }
}