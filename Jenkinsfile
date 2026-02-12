pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "lokeshs2612/my-app"
        K3S_SERVER   = "ubuntu@175.41.153.13" // K3s server IP from Terraform output
        K3S_PATH     = "/etc/rancher/k3s/k3s.yaml" // K3s kubeconfig path on server
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/lokeshsomasundaram/my-app.git'
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

        stage('Push Docker Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE:latest'
            }
        }

        stage('Get kubeconfig from K3s Server') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'k3s-ssh', 
                                                   keyFileVariable: 'SSH_KEY', 
                                                   usernameVariable: 'SSH_USER')]) {
                    sh '''
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no $SSH_USER@$K3S_SERVER:$K3S_PATH ./k3s.yaml
                        export KUBECONFIG=./k3s.yaml
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    export KUBECONFIG=./k3s.yaml
                    kubectl set image deployment/my-app my-app=$DOCKER_IMAGE:latest --record
                    kubectl rollout status deployment/my-app
                '''
            }
        }
    }

    post {
        success {
            echo '✅ App deployed successfully to K3s!'
        }
        failure {
            echo '❌ App deployment failed!'
        }
    }
}
