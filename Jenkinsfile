pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "docker4241/my-app"
        K8S_NAMESPACE = "dev"
    }
    stages {
       
        stage('Build & Test') {
            agent {
                docker {
                    image 'python:3.9'
                    args '-v $PWD:/app -w /app'  // Added working directory
                }
            }
            steps {
                sh '''
                      python -m venv venv
            . venv/bin/activate
            pip install -r requirements.txt
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-id') {
                        docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl config use-context your-k8s-context
                    kubectl set image deployment/my-app \\
                    app=${DOCKER_IMAGE}:${BUILD_NUMBER} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
    }
}
