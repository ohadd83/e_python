pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-cicd-app"
        CONTAINER_NAME = "python-app"
        APP_PORT = "5000"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

//        stage('Checkout') {
  //          steps {
    //            echo "Checking out source code..."

      //          git branch: 'main',
         //           url: 'https://github.com/YOUR_USERNAME/python-cicd-app.git'
           // }
    //    }

        stage('Test') {
            steps {
                echo "Running tests..."

                sh '''
                    python3 -m pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."

                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest \
                        .
                '''
            }
        }

        stage('Docker Image Test') {
            steps {
                echo "Testing Docker image..."

                sh '''
                    docker run -d \
                        --name ${CONTAINER_NAME}-test \
                        -p 5001:5000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''

                sleep 3

                curl -f http://localhost:5001

                docker stop ${CONTAINER_NAME}-test
                docker rm ${CONTAINER_NAME}-test
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying application..."

                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:5000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "Checking production application..."

                sh '''
                    sleep 3

                    curl -f http://localhost:${APP_PORT}/health
                '''
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "Deployment successful!"
            echo "Image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "Pipeline failed!"
            echo "======================================"
        }
    }
}

