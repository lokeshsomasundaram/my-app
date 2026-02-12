pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "lokeshs2612/my-app"
    }

    stages {

        stage('Clone App Code') {
            steps {
                git branch: 'main', url: 'https://github.com/lokeshsomasundaram/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    COMMIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    sh "docker build -t ${DOCKER_IMAGE}:${COMMIT_HASH} ."
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
                sh "docker push ${DOCKER_IMAGE}:${COMMIT_HASH}"
            }
        }

        stage('Deploy to k3s') {
            steps {
                // Retrieve kubeconfig from Infra pipeline
                unstash 'k3s-config'

                sh '''
                    export KUBECONFIG=$WORKSPACE/k3s.yaml
                    kubectl set image deployment/my-app my-app=${DOCKER_IMAGE}:${COMMIT_HASH} --record
                    kubectl rollout status deployment/my-app
                '''
            }
        }
    }

    post {
        success {
            echo '✅ App deployed successfully!'
        }
        failure {
            echo '❌ App deployment failed!'
        }
    }
}
