pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "lokeshs2612/my-app:latest"
        K3S_SERVER   = "52.221.249.37"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
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
                        scp -o StrictHostKeyChecking=no deployment.yaml ubuntu@$K3S_SERVER:/home/ubuntu/

                        ssh -o StrictHostKeyChecking=no ubuntu@$K3S_SERVER "
                            kubectl apply -f /home/ubuntu/deployment.yaml
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Application deployed successfully!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
