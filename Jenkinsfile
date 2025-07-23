pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-southeast-4'
        ECR_REPO = '686255978515.dkr.ecr.ap-southeast-4.amazonaws.com/python-flask-sample-app'
        DOCKER_IMAGE = "${ECR_REPO}:latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/kansari2302/python-flask-sample-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t python-flask-sample-app .'
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh 'docker tag python-flask-sample-app:latest $DOCKER_IMAGE'
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''
                    aws eks update-kubeconfig --name my-cluster --region $AWS_REGION
                    kubectl apply -f k8s/deployment.yaml
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
