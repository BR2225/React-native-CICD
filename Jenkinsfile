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
        GCLOUD_PATH = '/opt/google-cloud-sdk/bin' // Change this path to match your Linux agent's gcloud path
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${GITHUB_BRANCH}", url: "${GITHUB_REPO}"
            }
        }

        stage('Check Docker') {
            steps {
                script {
                    sh 'docker --version'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:$BUILD_NUMBER ."
                    sh "docker images"
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
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push $DOCKER_IMAGE:$BUILD_NUMBER
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    sh '''
                        export PATH=$PATH:$GCLOUD_PATH
                        gcloud auth login --brief || true
                        gcloud config set project $GKE_PROJECT_ID
                        gcloud container clusters get-credentials $GKE_CLUSTER_NAME --zone $GKE_ZONE
                        kubectl apply -f k8s/$DEPLOYMENT_FILE
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh "kubectl get pods"
                }
            }
        }
    }

    post {
        success {
            sh 'echo Successfully deployed to GKE'
        }
        failure {
            sh 'echo Build failed. Check logs.'
        }
        always {
            script {
                sh '''
                    export PATH=$PATH:$GCLOUD_PATH
                    gcloud auth revoke --all || true
                '''
            }
        }
    }
}
