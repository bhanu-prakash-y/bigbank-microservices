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
                    /var/lib/jenkins/.local/bin/aws ecr get-login-password \
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
        stage('Deploy to DEV') {
            steps {
                sh '''
            echo "Deploying Discovery Service image ${IMAGE_TAG} to DEV..."

            /var/lib/jenkins/.local/bin/aws eks update-kubeconfig \
                --region ${AWS_REGION} \
                --name microservices-cluster

            sed "s/IMAGE_TAG/${IMAGE_TAG}/g" \
                kubernetes/dev/deployment.yaml > deployment-rendered.yaml

            kubectl apply -f deployment-rendered.yaml
            kubectl apply -f kubernetes/dev/service.yaml

            kubectl rollout status \
                deployment/discovery-service \
                -n dev \
                --timeout=180s
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
