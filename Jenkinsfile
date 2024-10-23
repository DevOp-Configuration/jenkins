pipeline {
    agent any

    environment {
        // Define any environment variables
        DOCKER_IMAGE_NAME = "nextjs-app"
        DOCKER_CONTAINER_NAME = "nextjs-app-container"
        DEPLOY_PORT = "3000"  // Port on which the Next.js app will run
        ZIP_FILE_PATH = "/home/kosal/fileprojectupload/Archive.zip" // Path to the uploaded ZIP file on the Jenkins server
        UNZIPPED_DIR = "${env.WORKSPACE}/unzipped-app"
    }

    stages {
        stage('Unzip File') {
            steps {
                script {
                    // Unzip the uploaded ZIP file into the workspace directory
                    echo "Unzipping file..."
                    sh 'unzip -o ${ZIP_FILE_PATH} -d ${UNZIPPED_DIR}'
                    sh 'ls'
                }
            }
        }

        stage('Create Dockerfile') {
            steps {
                script {
                    // Create a Dockerfile in the unzipped directory
                    echo "Creating Dockerfile..."
                    writeFile file: "${UNZIPPED_DIR}/Dockerfile", text: """
                    # Base image
                    FROM node:18-alpine

                    # Set working directory
                    WORKDIR /app

                    # Copy package.json and package-lock.json
                    COPY package*.json ./

                    # Install dependencies
                    RUN yarn install --only=production

                    # Copy the rest of the application files
                    COPY . .

                    # Build the Next.js application
                    RUN yarn run build

                    # Expose the port that Next.js will run on
                    EXPOSE 3000

                    # Start the Next.js app
                    CMD ["yarn", "start"]
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    // Build the Docker image from the unzipped directory
                    sh """
                        cd ${UNZIPPED_DIR}
                        docker build -t ${DOCKER_IMAGE_NAME}:latest .
                    """
                }
            }
        }

        stage('Stop and Remove Existing Container') {
            steps {
                script {
                    // Stop and remove any existing container with the same name
                    echo "Stopping and removing existing container (if any)..."
                    sh """
                        docker stop ${DOCKER_CONTAINER_NAME} || true
                        docker rm ${DOCKER_CONTAINER_NAME} || true
                    """
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    echo "Running Docker container..."
                    // Run the new container with the Next.js app
                    sh """
                        docker run -d --name ${DOCKER_CONTAINER_NAME} -p ${DEPLOY_PORT}:${DEPLOY_PORT} ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Check Deployment') {
            steps {
                script {
                    // Check that the app is running and listening on the correct port
                    echo "Verifying that the app is running..."
                    sh """
                        docker logs ${DOCKER_CONTAINER_NAME}
                        curl -I http://localhost:${DEPLOY_PORT}
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up after the build
            echo "Cleaning up workspace..."
            cleanWs()
        }

        success {
            echo 'Deployment completed successfully!'
        }

        failure {
            echo 'Deployment failed.'
        }
    }
}
