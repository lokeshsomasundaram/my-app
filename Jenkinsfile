pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "lokeshs2612/my-app"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/lokeshsomasundaram/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Use Git commit hash for a unique tag
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = commitHash
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub', 
                    usernameVariable: 'USERNAME', 
                    passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                    export KUBECONFIG=${KUBECONFIG_FILE}
                    kubectl set image deployment/my-app my-app=${DOCKER_IMAGE}:${IMAGE_TAG} --record
                    kubectl rollout status deployment/my-app
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ App deployed successfully with image tag: ${IMAGE_TAG}"
        }
        failure {
            echo "❌ App deployment failed!"
        }
    }
}
