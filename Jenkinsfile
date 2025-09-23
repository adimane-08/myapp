pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "adimane0801/myapp" // Replace with your DockerHub repo
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

        stage('Validate YAML') {
            steps {
                // Fails if any tabs are found in k8s-deployment.yaml
                script {
                    bat '''
                    findstr /R "\\t" k8s-deployment.yaml && (echo Tabs found in YAML! && exit 1) || echo No tabs, YAML is clean
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t %DOCKER_IMAGE%:v1 ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%:v1
                        """
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

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check errors above."
        }
    }
}
