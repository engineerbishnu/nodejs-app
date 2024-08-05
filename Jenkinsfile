
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "nodejs-app"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials" // Replace with your Docker Hub credentials ID
        DOCKER_REGISTRY = "hub.docker.com"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    // Build Docker image
                    dockerImage = docker.build("${env.DOCKER_IMAGE}:${env.BUILD_ID}")
                    
                    // Tag and push Docker image
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", "${env.DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    // Run tests inside Docker container
                    dockerImage.inside {
                        // Set execute permissions for node_modules/.bin
                        sh 'chmod +x node_modules/.bin/mocha'
                        sh 'npm test'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes
                    kubeconfig(credentialsId: 'kubeconfig-id', serverUrl: 'https://0.0.0.0:45685') {
                        sh 'kubectl apply -f kubernetes-deployment.yaml'
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}

