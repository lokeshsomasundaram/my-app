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

        stage('Restore kubeconfig') {
            steps {
                // Restore kubeconfig stashed from Infra pipeline
                unstash 'k3s-config'
                sh 'mkdir -p $HOME/.kube'
                sh 'cp k3s.yaml $HOME/.kube/config'
                sh 'chmod 600 $HOME/.kube/config'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
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

        stage('Push Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // This deployment uses the kubeconfig restored above
                sh 'kubectl set image deployment my-app my-app=$DOCKER_IMAGE:latest --record'
                sh 'kubectl rollout status deployment my-app'
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
