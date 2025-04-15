
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sha256:af0568912cb661d87a8e38b015f3c3c6194f2a331dd6d7530103398a1ff0c321'
        DOCKER_TAG = '16'
        GITHUB_REPO = 'https://github.com/BR2225/React-native-CICD.git'
        GITHUB_BRANCH = 'main'
        // GKE Configuration
        GKE_PROJECT_ID = 'inductive-gift-456306-h4'
        GKE_CLUSTER_NAME = 'react-native-cluster-1'
        GKE_ZONE = 'us-central1-a'
        DEPLOYMENT_NAME = 'react-native-app'
        CONTAINER_PORT = '19000'

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${GITHUB_BRANCH}", url: "${GITHUB_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    script {
                        bat "docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
                        bat "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        bat "docker logout"
                    }
                }
            }
        }
    }

        stage('Deploy to GKE') {
    steps {
        script {
            // Authenticate using gcloud (login with your user credentials)
            bat "gcloud auth login"

            // Set the project in gcloud configuration
            bat "gcloud config set project ${GKE_PROJECT_ID}"

            // Get credentials for your GKE cluster
            bat "gcloud container clusters get-credentials ${GKE_CLUSTER_NAME} --zone ${GKE_ZONE}"

            // Apply Kubernetes manifests (assuming the deployment.yaml file is inside the k8s folder)
            bat "kubectl apply -f k8s/deployment.yaml"
        }
    }
}


    post {
        success {
            echo "✅ Successfully deployed %DOCKER_IMAGE%:%DOCKER_TAG% to GKE"
        }
        failure {
            echo "❌ Build failed. Check logs."
        }
        always {
            // Clean up GCloud authentication
            bat "gcloud auth revoke --all"
        }
    }
}
