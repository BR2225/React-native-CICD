pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'br2225/react-native-app'
        DOCKER_TAG = 'v1.0.0'
        GITHUB_REPO = 'https://github.com/BR2225/React-native-CICD.git'
        GITHUB_BRANCH = 'main'

        // GKE Configuration
        GKE_PROJECT_ID = 'inductive-gift-456306-h4'
        GKE_CLUSTER_NAME = 'react-native-cluster-1'
        GKE_ZONE = 'us-central1-a'
        DEPLOYMENT_NAME = 'react-native-app'
        CONTAINER_PORT = '19000'
    }

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
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        sh "docker logout"
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    sh "gcloud auth login"
                    sh "gcloud config set project ${GKE_PROJECT_ID}"
                    sh "gcloud container clusters get-credentials ${GKE_CLUSTER_NAME} --zone ${GKE_ZONE}"
                    sh "kubectl apply -f k8s/deployment.yaml"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Successfully deployed ${DOCKER_IMAGE}:${DOCKER_TAG} to GKE"
        }
        failure {
            echo "❌ Build failed. Check logs."
        }
        always {
            script {
                sh "gcloud auth revoke --all || true"
            }
        }
    }
}
