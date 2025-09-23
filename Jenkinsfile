pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')  // Jenkins credentials ID
        DOCKER_IMAGE = "adimane0801/myapp"
    }
    options {
                skipDefaultCheckout(true) // Required to clean before default checkout
            }

    stages {
        stage('Clean Workspace') {
            steps {
                // Deletes old files in Jenkins workspace
                cleanWs()
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/adimane-08/myapp.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:v1")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        docker.image("${DOCKER_IMAGE}:v1").push()
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        bat 'kubectl apply -f k8s-deployment.yaml --validate=false || echo "Deployment file not found"'
                    }
                }
            }
        }
    }
}
