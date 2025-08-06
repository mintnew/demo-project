pipeline {
    agent any

    // Define environment variables at the top level
    // This makes them accessible throughout the entire pipeline, including the post block.
    environment {
        // Use the correct Docker Hub registry URL. 
        // 'docker.io' is the correct host for the default public registry.
        DOCKER_REGISTRY = 'docker.io'
        APP_NAME = 'my-nodejs-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBECONFIG_ID = 'kubeconfig-file'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git url: 'https://github.com/<your-github-username>/<your-repo-name>.git',
                    credentialsId: 'your-github-ssh-credentials-id'
            }
        }

        stage('Build Docker Image') {
            steps {
                // We're now using the corrected DOCKER_REGISTRY variable.
                sh "docker build -t ${env.DOCKER_REGISTRY}/${env.DOCKER_USERNAME}/${env.APP_NAME}:${env.IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                // The withCredentials block is the correct and secure way to handle credentials.
                // It injects username/password into temporary environment variables.
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials',
                                                       usernameVariable: 'DOCKER_USER',
                                                       passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin ${env.DOCKER_REGISTRY}"
                        sh "docker push ${env.DOCKER_REGISTRY}/${DOCKER_USER}/${env.APP_NAME}:${env.IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // The withKubeConfig step securely handles the kubeconfig file.
                script {
                    withKubeConfig(credentialsId: env.KUBECONFIG_ID) {
                        sh "kubectl set image deployment/${env.APP_NAME}-deployment ${env.APP_NAME}=${env.DOCKER_REGISTRY}/${env.DOCKER_USERNAME}/${env.APP_NAME}:${env.IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Access global environment variables using the 'env' prefix
                // and a separate withCredentials block for secure cleanup.
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials',
                                                   usernameVariable: 'DOCKER_USER',
                                                   passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker rmi -f ${env.DOCKER_REGISTRY}/${DOCKER_USER}/${env.APP_NAME}:${env.IMAGE_TAG}"
                    // It's generally not recommended to push 'latest' tags in a CI/CD pipeline,
                    // as it can lead to confusion. A numbered tag is more reliable.
                    // If you still want to do this, use a separate tag in the build stage.
                    // sh "docker rmi -f ${env.DOCKER_REGISTRY}/${DOCKER_USER}/${env.APP_NAME}:latest" 
                }
            }
        }
        success {
            echo 'Pipeline finished successfully! 👍'
        }
        failure {
            echo 'Pipeline failed! 👎'
        }
    }
}
