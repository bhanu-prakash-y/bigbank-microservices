pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '255248181810'
        ECR_REPOSITORY = 'bigbank/discovery-service'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "Checking workspace..."
                    pwd
                    ls -la

                    test -f discovery-0.0.1.jar
                    test -f Dockerfile
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    /usr/local/bin/aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${ECR_REPOSITORY}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Docker Tag') {
            steps {
                sh '''
                    docker tag \
                    ${ECR_REPOSITORY}:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    docker push \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo "Discovery image ${IMAGE_TAG} pushed successfully to ECR."
        }

        failure {
            echo 'Discovery CI pipeline failed.'
        }
    }
}