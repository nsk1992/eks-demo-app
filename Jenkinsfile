pipeline {

    agent {
        label 'agent'
    }

    options {
        timestamps()
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPOSITORY = 'eks-demo-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=eks-demo-app \
                      -Dsonar.projectName=eks-demo-app \
                      -Dsonar.sources=. \
                      -Dsonar.python.version=3
                    '''
                }
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
                docker build -t eks-demo-app:${GIT_SHA} .
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
                --region ${AWS_REGION} | \
                docker login \
                --username AWS \
                --password-stdin \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                ACCOUNT_ID=$(aws sts get-caller-identity \
                --query Account \
                --output text)

                IMAGE_URI=${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}

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
        always {
            sh 'docker image prune -f || true'
        }
    }
}
