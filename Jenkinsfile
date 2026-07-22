pipeline {
    agent any

    environment {
        AWS_REGION        = 'us-east-1'
        ECR_REGISTRY      = '264450777137.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO          = 'devops'
        IMAGE_TAG         = "${env.BUILD_NUMBER}"
        FULL_IMAGE        = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Shreeram-cloud/aws-eks-deployment.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}

                        docker push ${FULL_IMAGE}

                        # Also tag as latest
                        docker tag ${FULL_IMAGE} ${ECR_REGISTRY}/${ECR_REPO}:latest
                        docker push ${ECR_REGISTRY}/${ECR_REPO}:latest
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh """
                    docker rmi ${FULL_IMAGE} || true
                    docker rmi ${ECR_REGISTRY}/${ECR_REPO}:latest || true
                """
            }
        }
    }

    post {
        success {
            echo "Build ${IMAGE_TAG} pushed to ECR successfully"
        }
        failure {
            echo "Pipeline failed — check the logs above"
        }
        always {
            cleanWs()
        }
    }
}
