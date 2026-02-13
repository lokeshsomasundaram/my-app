pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "lokeshs2612/my-app:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/lokeshsomasundaram/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                usernameVariable: 'USER',
                                passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to K3s') {
            steps {
                sshagent(['k3s-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@52.221.249.37 \
                        "kubectl set image deployment/my-app my-app=$DOCKER_IMAGE"
                    '''
                }
            }
        }
    }
}
