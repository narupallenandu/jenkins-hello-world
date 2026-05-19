pipeline {
    agent any

    environment {

        BUILD_DIR = "build"

        GIT_REPO_URL = "https://github.com/narupallenandu/jenkins-hello-world.git"

        IMAGE_NAME = "nandu-world"

        IMAGE_TAG = "v1"

        DOCKER_HUB_USERNAME = "narupallenandu"

        CONTAINER_NAME = "demo-hello-world-container"
    }

    parameters {

        string(
            name: 'GIT_BRANCH',
            defaultValue: 'main',
            description: 'Git Branch Name'
        )
    }

    stages {

        // -------------------------------------------------------------------
        // Stage 1: Create Build Directory
        // -------------------------------------------------------------------
        stage('Create Build Directory') {

            steps {

                sh """
                    mkdir -p ${WORKSPACE}/${BUILD_DIR}
                """
            }
        }

        // -------------------------------------------------------------------
        // Stage 2: Checkout Repository
        // -------------------------------------------------------------------
        stage('Checkout') {

            steps {

                dir("${BUILD_DIR}") {

                    git url: "${GIT_REPO_URL}",
                        branch: "${params.GIT_BRANCH}"
                }
            }
        }

        // -------------------------------------------------------------------
        // Stage 3: Build Docker Image
        // -------------------------------------------------------------------
        stage('Build Docker Image') {

            steps {

                dir("${BUILD_DIR}") {

                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        // -------------------------------------------------------------------
        // Stage 4: Stop Old Container
        // -------------------------------------------------------------------
        stage('Stop Old Container') {

            steps {

                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                """
            }
        }

        // -------------------------------------------------------------------
        // Stage 5: Deploy Container
        // -------------------------------------------------------------------
        stage('Deploy Container') {

            steps {

                sh """
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p 9006:80 \
                    ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        // -------------------------------------------------------------------
        // Stage 6: Verify Deployment
        // -------------------------------------------------------------------
        stage('Verify Deployment') {

            steps {

                sh """
                    docker ps
                """
            }
        }

        // -------------------------------------------------------------------
        // Stage 7: Syft Scan
        // -------------------------------------------------------------------
        stage('Syft Scan') {

            steps {

                sh """
                    syft ${IMAGE_NAME}:${IMAGE_TAG} -o table

                    syft ${IMAGE_NAME}:${IMAGE_TAG} \
                    -o json > syft-report.json
                """
            }
        }

        // -------------------------------------------------------------------
        // Stage 8: Grype Scan
        // -------------------------------------------------------------------
        stage('Grype Scan') {

            steps {

                sh """
                    grype ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        // -------------------------------------------------------------------
        // Stage 9: Push Image To DockerHub
        // -------------------------------------------------------------------
        stage('Push Image To DockerHub') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'nandu-world',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push \
                        ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {

        success {

            echo 'Application deployed successfully 🚀'
        }

        failure {

            echo 'Pipeline failed ❌'
        }
    }
}
