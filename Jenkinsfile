pipeline {
    agent any

    environment {
        // Define your environment variables
        REPO_URL = 'https://github.com/yourusername/your-repo.git'
        DOCKER_REGISTRY = 'your-docker-registry' // e.g., 'docker.io/yourusername'
        IMAGE_NAME = 'python-app'
        TAG = 'latest' // or use ${env.BUILD_NUMBER} for unique tags
        FULL_IMAGE_NAME = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${TAG}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git REPO_URL
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    sh "docker tag ${IMAGE_NAME}:${TAG} ${FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Login to Docker Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', passwordVariable: 'REGISTRY_PASS', usernameVariable: 'REGISTRY_USER')]) {
                        sh "echo $REGISTRY_PASS | docker login ${DOCKER_REGISTRY} -u $REGISTRY_USER --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push ${FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                        // Apply Kubernetes manifests from the k8s directory
                        sh 'kubectl apply -f k8s/'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            // Add steps to clean up if necessary, such as logging out of the Docker registry
            sh "docker logout ${DOCKER_REGISTRY}"
        }
    }
}
