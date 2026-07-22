pipeline {
    agent any

    environment {
        AWS_REGION        = 'us-east-1'
        ECR_REGISTRY      = '264450777137.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO          = 'devops'
        IMAGE_TAG         = "${env.BUILD_NUMBER}"
        FULL_IMAGE        = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
        GITHUB_TOKEN      = credentials('github-token')
        GITOPS_REPO       = 'https://github.com/Shreeram-cloud/k8s-deploy-arcocd.git'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
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
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker push ${FULL_IMAGE}

                    docker tag ${FULL_IMAGE} ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Update GitOps Repo') {
            steps {
                sh """
                    # Clone the GitOps repo
                    git clone https://${GITHUB_TOKEN}@github.com/Shreeram-cloud/k8s-deploy-arcocd.git gitops

                    # Update image tag in deployment.yaml
                    sed -i 's|${ECR_REGISTRY}/${ECR_REPO}:.*|${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}|g' gitops/deployment.yaml

                    # Commit and push
                    cd gitops
                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins"
                    git add deployment.yaml
                    git commit -m "ci: update image tag to ${IMAGE_TAG}"
                    git push origin master
                """
            }
        }

        stage('Cleanup') {
            steps {
                sh """
                    docker rmi ${FULL_IMAGE} || true
                    rm -rf gitops
                """
            }
        }
    }

    post {
        success {
            echo "Build ${IMAGE_TAG} deployed — GitOps repo updated"
        }
        failure {
            echo "Pipeline failed — check the logs above"
        }
        always {
            cleanWs()
        }
    }
}
