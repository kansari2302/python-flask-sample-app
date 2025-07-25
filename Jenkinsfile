pipeline {
    agent any

    parameters {
        choice(name: 'STAGE_TO_RUN', choices: ['ALL', 'CHECKOUT', 'BUILD', 'PUSH', 'DEPLOY'], description: 'Select stage to run')
    }

    environment {
        APP_NAME = "python-flask-sample-app"
        ACCOUNT_ID = '686255978515'
        IMAGE_TAG = "${APP_NAME}:${BUILD_NUMBER}"
        AWS_REGION = 'ap-southeast-4'
        ECR_REPO = '686255978515.dkr.ecr.ap-southeast-4.amazonaws.com/python-flask-sample-app'
        DOCKER_IMAGE = "${ECR_REPO}:latest"
    }

    stages {
        // stage('Checkout Code') {
        //     when {
        //         anyOf {
        //             expression { params.STAGE_TO_RUN == 'ALL' }
        //             expression { params.STAGE_TO_RUN == 'CHECKOUT' }
        //         }
        //     }
        //     steps {
        //         git branch: 'master', url: 'https://github.com/kansari2302/python-flask-sample-app.git'
        //     }
        // }

        stage('Build Docker Image') {
            when {
                anyOf {
                    expression { params.STAGE_TO_RUN == 'ALL' }
                    expression { params.STAGE_TO_RUN == 'BUILD' }
                }
            }
            steps {
                echo "Building Docker image ${IMAGE_TAG}"
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        stage('Login to ECR') {
            when {
                anyOf {
                    expression { params.STAGE_TO_RUN == 'ALL' }
                    expression { params.STAGE_TO_RUN == 'PUSH' }
                }
            }
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    echo "Logging in to ECR..."
                    sh '''
                        aws sts get-caller-identity
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            when {
                anyOf {
                    expression { params.STAGE_TO_RUN == 'ALL' }
                    expression { params.STAGE_TO_RUN == 'PUSH' }
                }
            }
            steps {
                sh "docker tag ${IMAGE_TAG} ${DOCKER_IMAGE}"
                sh "docker push ${DOCKER_IMAGE}"
                echo "Docker image ${DOCKER_IMAGE} pushed to ECR"
            }
        }

        stage('Deploy to EKS') {
            when {
                anyOf {
                    expression { params.STAGE_TO_RUN == 'ALL' }
                    expression { params.STAGE_TO_RUN == 'DEPLOY' }
                }
            }
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
