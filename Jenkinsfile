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
                    bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
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

        stage('Deploy to GKE') {
            steps {
                withCredentials([file(credentialsId: 'gke-service-account-key', variable: 'GKE_KEY')]) {
                    script {
                        bat "gcloud auth activate-service-account --key-file=%GKE_KEY%"
                        bat "gcloud config set project %GKE_PROJECT_ID%"
                        bat "gcloud container clusters get-credentials %GKE_CLUSTER_NAME% --zone %GKE_ZONE%"
                        
                       
                        bat "kubectl apply -f deployment.yaml"
                    }
                }
            }
        }
    }

    post {
        success {
            bat 'echo "✅ Successfully deployed to GKE"'
        }
        failure {
            bat 'echo "❌ Build failed. Check logs."'
        }
        always {
            bat "gcloud auth revoke --all || exit 0"
        }
    }
}
