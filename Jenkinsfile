pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/react-native-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        GCLOUD_PROJECT_ID = 'your-gcp-project-id'
        GCLOUD_CLUSTER_NAME = 'your-cluster-name'
        GCLOUD_ZONE = 'your-zone'
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                        docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        
        stage('Deploy to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-credentials', variable: 'GCLOUD_CREDS')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=${GCLOUD_CREDS}
                        gcloud config set project ${GCLOUD_PROJECT_ID}
                        gcloud container clusters get-credentials ${GCLOUD_CLUSTER_NAME} --zone ${GCLOUD_ZONE}
                        kubectl set image deployment/react-native-app react-native-app=${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }
}