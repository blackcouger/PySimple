pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "docker4241/test-phython-app"
        KUBE_CONFIG = credentials('kube-config') // Jenkins credential with Kubeconfig 
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/blackcouger/PySimple.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-id') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f test-app-deployment
                        kubectl --kubeconfig=$KUBECONFIG set image deployment/test-app-deployment \
                            test-app-container=${DOCKER_IMAGE}:${BUILD_NUMBER} --record
                    """
                }
            }
        }
    }
}
