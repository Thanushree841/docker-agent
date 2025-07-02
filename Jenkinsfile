pipeline {
    agent { label'docker' }

    environment {
        DOCKER_REPO_CREDENTIALS = 'ade72d11-cc03-4d9c-957d-bba399bceb43'
        DOCKER_IMAGE = '9148092892/agent'          
        GIT_REPO = 'https://github.com/Thanushree841/docker-agent.git'
        BRANCH = 'main'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning repository from ${GIT_REPO}"
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    COMMIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    IMAGE_TAG_COMMIT = "${DOCKER_IMAGE}:${COMMIT_HASH}"
                    IMAGE_TAG_LATEST = "${DOCKER_IMAGE}:latest"

                    echo "Building Docker image with tags: ${IMAGE_TAG_COMMIT} and ${IMAGE_TAG_LATEST}"
                    sh "docker build -t ${IMAGE_TAG_COMMIT} -t ${IMAGE_TAG_LATEST} ."
                }
            }
        }

        stage('Authenticate with Docker Registry') {
            steps {
                script {
                    echo "Logging in to Docker registry"
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_REPO_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    echo "Pushing images to registry"
                    echo "IMAGE_TAG_COMMIT: ${IMAGE_TAG_COMMIT}"
                    echo "IMAGE_TAG_LATEST: ${IMAGE_TAG_LATEST}"
                    sh "docker push ${IMAGE_TAG_COMMIT}"
                    sh "docker push ${IMAGE_TAG_LATEST}"
                }
            }
        }
    }

    post {
        success {
            echo " Build and push completed successfully!"
        }
        failure {
            echo " Build or push failed. Check logs for details."
        }
        always {
            echo " Build finished: ${currentBuild.currentResult}"
        }
    }
}
