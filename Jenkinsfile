pipeline {

    agent {
        label 'agent'
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify AWS Access') {
            steps {
                sh '''
                aws sts get-caller-identity
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.GIT_SHA = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }

                sh '''
                docker build \
                -t eks-demo-app:${GIT_SHA} .
                '''
            }
        }

        stage('Login ECR') {
            steps {
                sh '''
                ACCOUNT_ID=$(aws sts get-caller-identity \
                --query Account \
                --output text)

                aws ecr get-login-password \
                --region us-east-1 | \
                docker login \
                --username AWS \
                --password-stdin \
                ${ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                ACCOUNT_ID=$(aws sts get-caller-identity \
                --query Account \
                --output text)

                IMAGE_URI=${ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/eks-demo-app

                docker tag \
                eks-demo-app:${GIT_SHA} \
                ${IMAGE_URI}:${GIT_SHA}

                docker push \
                ${IMAGE_URI}:${GIT_SHA}
                '''
            }
        }
    }

    post {
        success {
            echo 'Image pushed successfully'
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
