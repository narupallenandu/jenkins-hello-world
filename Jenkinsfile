pipeline {

    agent any

    environment {

        // ----------------------------------------------------------------
        // Application Details
        // ----------------------------------------------------------------
        APP_NAME            = "nandu-world"
        IMAGE_NAME          = "nandu-world"
        IMAGE_TAG           = "v1"

        // ----------------------------------------------------------------
        // DockerHub Details
        // ----------------------------------------------------------------
        DOCKER_HUB_USERNAME = "narupallenandu"

        // ----------------------------------------------------------------
        // Container Details
        // ----------------------------------------------------------------
        CONTAINER_NAME      = "demo-hello-world-container"

        // ----------------------------------------------------------------
        // Git Details
        // ----------------------------------------------------------------
        GIT_REPO_URL        = "https://github.com/narupallenandu/jenkins-hello-world.git"

        // ----------------------------------------------------------------
        // Workspace
        // ----------------------------------------------------------------
        BUILD_DIR           = "build"
    }

    parameters {

        string(
            name: 'GIT_BRANCH',
            defaultValue: 'main',
            description: 'Enter Git Branch Name'
        )
    }

    options {

        timestamps()

        disableConcurrentBuilds()
    }

    stages {

        // ================================================================
        // Stage 1 : Prepare Workspace
        // ================================================================
        stage('Prepare Workspace') {

            steps {

                echo "Creating build directory..."

                sh """
                    rm -rf ${BUILD_DIR}

                    mkdir -p ${BUILD_DIR}
                """
            }
        }

        // ================================================================
        // Stage 2 : Clone Repository
        // ================================================================
        stage('Checkout Code') {

            steps {

                echo "Cloning GitHub repository..."

                dir("${BUILD_DIR}") {

                    git(
                        url: "${GIT_REPO_URL}",
                        branch: "${params.GIT_BRANCH}"
                    )
                }
            }
        }

        // ================================================================
        // Stage 3 : Build Docker Image
        // ================================================================
        stage('Build Docker Image') {

            steps {

                echo "Building Docker image..."

                dir("${BUILD_DIR}") {

                    sh """
                        docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        // ================================================================
        // Stage 4 : Stop Existing Container
        // ================================================================
        stage('Stop Existing Container') {

            steps {

                echo "Stopping old container if running..."

                sh """
                    docker stop ${CONTAINER_NAME} || true

                    docker rm ${CONTAINER_NAME} || true
                """
            }
        }

        // ================================================================
        // Stage 5 : Deploy Container
        // ================================================================
        stage('Deploy Application') {

            steps {

                echo "Deploying Docker container..."

                sh """
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p 9006:80 \
                    ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        // ================================================================
        // Stage 6 : Verify Deployment
        // ================================================================
        stage('Verify Deployment') {

            steps {

                echo "Checking running containers..."

                sh """
                    docker ps
                """
            }
        }

        // ================================================================
        // Stage 7 : Generate SBOM using Syft
        // ================================================================
        stage('Syft Scan') {

            steps {

                echo "Generating SBOM report..."

                sh """
                    syft ${IMAGE_NAME}:${IMAGE_TAG} -o table

                    syft ${IMAGE_NAME}:${IMAGE_TAG} \
                    -o json > syft-report.json
                """
            }
        }

        // ================================================================
        // Stage 8 : Vulnerability Scan using Grype
        // ================================================================
        stage('Grype Scan') {

            steps {

                echo "Running vulnerability scan..."

                sh """
                    grype ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        // ================================================================
        // Stage 9 : Push Image to DockerHub
        // ================================================================
        stage('Push Image To DockerHub') {

            steps {

                echo "Logging into DockerHub..."

                withCredentials([
                    usernamePassword(
                        credentialsId: 'nandu-world',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh """
                        echo \$DOCKER_PASS | \
                        docker login -u \$DOCKER_USER --password-stdin

                        docker tag \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push \
                        ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {

        // ================================================================
        // SUCCESS
        // ================================================================
        success {

            echo 'Application deployed successfully 🚀'
        }

        // ================================================================
        // FAILURE
        // ================================================================
        failure {

            echo 'Pipeline failed ❌'
        }

        // ================================================================
        // ALWAYS
        // ================================================================
        always {

            echo 'Cleaning up Docker login session...'

            sh 'docker logout || true'
        }
    }
}
