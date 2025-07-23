pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-southeast-4'
        ECR_REPO = 'sample-flask-app'
        IMAGE_TAG = 'latest'
        ACCOUNT_ID = '686255978515'
        ECR_URL = "${ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/kansari2302/python-flask-sample-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | \
                        docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                script {
                    sh '''
                        docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URL}:${IMAGE_TAG}
                        docker push ${ECR_URL}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@16.26.96.174 \
                        "docker pull ${ECR_URL}:${IMAGE_TAG} && \
                        docker stop flask-app || true && docker rm flask-app || true && \
                        docker run -d --name flask-app -p 5000:5000 ${ECR_URL}:${IMAGE_TAG}"
                    '''
                }
            }
        }
    }
}
