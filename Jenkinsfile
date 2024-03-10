pipeline {
    agent any

    environment {
        // Define your Docker image name here
        DOCKER_IMAGE = 'saadatkhan/survey645'
        // Define the version/tag. Using Jenkins build number for uniqueness
        VERSION = "${BUILD_NUMBER}"
    }

    stages {
          stage('Prepare Application') {
            steps {
                script {
                    // Clean up any previous .war file
                    sh 'rm -f survey645.war'
                    
                    // Package a new .war file
                    // Assuming all files needed for the war are in the root directory
                    sh 'jar -cvf survey645.war *'
                }
            }
          }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build --no-cache -t ${DOCKER_IMAGE}:${VERSION} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "echo ${DOCKERHUB_PASSWORD} | docker login --username ${DOCKERHUB_USERNAME} --password-stdin"
                    }
                    // Push the Docker image
                    sh "docker push ${DOCKER_IMAGE}:${VERSION}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Assuming you have a deployment.yaml that needs the image to be set
                    // Replace the image in the Kubernetes deployment with the new version
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh """
                           kubectl set image deployment/survey645-deployment survey645=${DOCKER_IMAGE}:${VERSION} --record
                           """
                    }
                }
            }
        }
    }
    post {
        always {
            // Optionally clean up Docker images to save space
            sh "docker rmi ${DOCKER_IMAGE}:${VERSION}"
        }
    }
}
