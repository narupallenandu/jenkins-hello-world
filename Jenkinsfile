pipeline {

    agent any

    environment {

        // ------------------------------------------------------------
        // Application Details
        // ------------------------------------------------------------
        IMAGE_NAME = "nandu-world"
        IMAGE_TAG  = "v1"

        // ------------------------------------------------------------
        // DockerHub Details
        // ------------------------------------------------------------
        DOCKER_HUB_USERNAME = "narupallenandu"

        // ------------------------------------------------------------
        // Container Details
        // ------------------------------------------------------------
        CONTAINER_NAME = "demo-hello-world-container"

        // ------------------------------------------------------------
        // Git Repository
        // ------------------------------------------------------------
        GIT_REPO_URL = "https://github.com/narupallenandu/jenkins-hello-world.git"

        BUILD_DIR = "build"
    }

    parameters {

        string(
            name: 'GIT_BRANCH',
            defaultValue: 'main',
            description: 'Enter Git Branch'
        )
    }

    options {

        timestamps()

        disableConcurrentBuilds()
    }

    stages {

        // ============================================================
        // Stage 1 : Prepare Workspace
        // ============================================================
        stage('Prepare Workspace') {

            steps {

                echo "Preparing workspace..."

                sh """
                    rm -rf ${BUILD_DIR}

                    mkdir -p ${BUILD_DIR}
                """
            }
        }

        // ============================================================
        // Stage 2 : Checkout Source Code
        // ============================================================
        stage('Checkout Code') {

            steps {

                echo "Cloning Git repository..."

                dir("${BUILD_DIR}") {

                    git(
                        url: "${GIT_REPO_URL}",
                        branch: "${params.GIT_BRANCH}"
                    )
                }
            }
        }

        // ============================================================
        // Stage 3 : Build Docker Image
        // ============================================================
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

        // ============================================================
        // Stage 4 : Stop Old Container
        // ============================================================
        stage('Stop Old Container') {

            steps {

                echo "Stopping old container..."

                sh """
                    docker stop ${CONTAINER_NAME} || true

                    docker rm ${CONTAINER_NAME} || true
                """
            }
        }

        // ============================================================
        // Stage 5 : Deploy Container
        // ============================================================
        stage('Deploy Container') {

            steps {

                echo "Deploying application container..."

                sh """
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p 9006:80 \
                    ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        // ============================================================
        // Stage 6 : Verify Deployment
        // ============================================================
        stage('Verify Deployment') {

            steps {

                echo "Verifying deployment..."

                sh "docker ps"
            }
        }

        // ============================================================
        // Stage 7 : Syft Scan
        // ============================================================
        stage('Syft Scan') {

            steps {

                echo "Generating SBOM using Syft..."

                sh """
                    syft ${IMAGE_NAME}:${IMAGE_TAG} -o table

                    syft ${IMAGE_NAME}:${IMAGE_TAG} \
                    -o json > syft-report.json
                """
            }
        }

        // ============================================================
        // Stage 8 : Grype Scan
        // ============================================================
        stage('Grype Scan') {

            steps {

                echo "Running vulnerability scan using Grype..."

                sh """
                    grype ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        // ============================================================
        // Stage 9 : Push Image To DockerHub
        // ============================================================
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

                    sh '''
                        echo "$DOCKER_PASS" | \
                        docker login -u "$DOCKER_USER" --password-stdin

                        docker tag \
                        nandu-world:v1 \
                        narupallenandu/nandu-world:v1

                        docker push \
                        narupallenandu/nandu-world:v1
                    '''
                }
            }
        }
    }

    post {

        success {

            echo 'Pipeline executed successfully 🚀'
        }

        failure {

            echo 'Pipeline execution failed ❌'
        }

        always {

            echo 'Logging out from DockerHub...'

            sh 'docker logout || true'
        }
    }
}
