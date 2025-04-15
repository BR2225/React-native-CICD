pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'baibhav225/react-native-app'
        GITHUB_REPO = 'https://github.com/BR2225/React-native-CICD.git'
        GITHUB_BRANCH = 'main'

        // GKE Configuration
        GKE_PROJECT_ID = 'inductive-gift-456306-h4'
        GKE_CLUSTER_NAME = 'react-native-cluster-1'
        GKE_ZONE = 'us-central1-a'
        DEPLOYMENT_FILE = 'deployment.yaml'
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
                    bat "docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% ."
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
                        bat """
                            echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                            docker push %DOCKER_IMAGE%:%BUILD_NUMBER%
                            docker logout
                        """
                    }
                }
            }
        }

               stage('Deploy to GKE') {
            environment {
                GCLOUD_PATH = "C:\\Program Files (x86)\\Google\\Cloud SDK\\google-cloud-sdk\\bin"
            }
            steps {
                script {
                    // Set Google Cloud SDK path
                    bat "SET PATH=%PATH%;${GCLOUD_PATH}"
                    
                   
                    
                    // Deploy to GKE
                    bat """
                        gcloud auth login
                        gcloud config set project inductive-gift-456306-h4
                        gcloud container clusters get-credentials react-native-cluster-1 --zone us-central1-a
                        kubectl apply -f deployment.yaml
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    bat "kubectl get pods"
                }
            }
        }
    }

    post {
        success {
            bat 'echo ✅ Successfully deployed to GKE'
        }
        failure {
            bat 'echo ❌ Build failed. Check logs.'
        }
        always {
            bat 'gcloud auth revoke --all || exit 0'
        }
    }
}
