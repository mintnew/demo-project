pipeline {
    agent any // Or a specific Docker agent if you want to isolate builds

    environment {
        DOCKER_REGISTRY = 'docker.io' // Or your private registry URL
        DOCKER_USERNAME = credentials('docker-hub-credentials').getUsername() // Using Jenkins Credentials
        DOCKER_PASSWORD = credentials('docker-hub-credentials').getPassword() // Using Jenkins Credentials
        APP_NAME = 'my-nodejs-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}" // Use Jenkins build number for unique tags
        KUBECONFIG_ID = 'kubeconfig-file' // ID of your Kubeconfig credential in Jenkins
        DOCKER_REGISTRY = 'https://hub.docker.com/repositories/saurabh0019'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git url: 'https://github.com/<your-github-username>/<your-repo-name>.git',
                    credentialsId: 'your-github-ssh-credentials-id' // Or leave if public repo
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${APP_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {

                    // Login and push the Docker image
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials',
                                                       usernameVariable: 'DOCKER_USER',
                                                       passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin ${DOCKER_REGISTRY}"
                        sh "docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${APP_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update the Kubernetes deployment with the new image tag
                    // Using 'withKubeConfig' if you stored kubeconfig as a file credential
                    withKubeConfig(credentialsId: KUBECONFIG_ID) {
                        sh "kubectl set image deployment/${APP_NAME}-deployment ${APP_NAME}=${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${APP_NAME}:${IMAGE_TAG}"
                    }
                    // Or if Jenkins is running within K8s and has service account permissions:
                    // sh "kubectl set image deployment/${APP_NAME}-deployment ${APP_NAME}=${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${APP_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Optional: Add steps to verify the deployment, e.g., curl the service endpoint
                    // For Minikube:
                    // sh "minikube service ${APP_NAME}-service --url"
                    // Or simple curl against the service in the cluster if Jenkins is inside K8s
                    sh "kubectl get pods -l app=${APP_NAME}"
                    sh "kubectl get svc ${APP_NAME}-service"
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker images (optional, to save space)
            script {
                sh "docker rmi -f ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${APP_NAME}:${IMAGE_TAG}"
                sh "docker rmi -f ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${APP_NAME}:latest" // If you also tag latest
            }
        }
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
