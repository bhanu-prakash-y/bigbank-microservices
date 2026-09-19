pipeline {
    agent any

   environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '255248181810'
        ECR_REPOSITORY = 'bigbank/discovery-service'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
        PATH = '/var/lib/jenkins/.local/bin:/usr/local/bin:/usr/bin:/bin'
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
                    aws ecr get-login-password \
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
        stage('Deploy to UAT using Helm') {
            steps {
                sh '''
            echo "Deploying Discovery Service image ${IMAGE_TAG} to UAT using Helm..."

            aws eks update-kubeconfig \
                --region ${AWS_REGION} \
                --name microservices-cluster

            helm upgrade --install discovery-service \
                helm/discovery-service \
                --namespace uat \
                --set image.tag=${IMAGE_TAG} \
                --wait \
                --timeout 5m
        '''
            }
        }

//         stage('Deploy to UAT') {
//             steps {
//                 sh '''
//                     echo "Deploying Discovery Service image ${IMAGE_TAG} to UAT..."

//                     aws eks update-kubeconfig \
//                     --region ${AWS_REGION} \
//                     --name microservices-cluster

//                     sed "s/IMAGE_TAG/${IMAGE_TAG}/g" \
//                     kubernetes/uat/deployment.yaml > deployment-rendered.yaml

//                     kubectl apply -f deployment-rendered.yaml
//                     kubectl apply -f kubernetes/uat/service.yaml

    //                     kubectl rollout status \
    //                     deployment/discovery-service \
    //                     -n uat \
    //                     --timeout=180s
    //                 '''
    //             }
    //         }
    }

    post {
        success {
            echo "Discovery image ${IMAGE_TAG} deployed successfully to UAT."
        }

        failure {
            echo 'Discovery CI/CD pipeline failed.'
        }
    }
}
