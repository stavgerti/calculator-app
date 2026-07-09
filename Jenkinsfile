pipeline {
    agent none

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "992382545251.dkr.ecr.us-east-1.amazonaws.com/stav-calculator-app"
    }

    stages {
        stage('Build Image') {
            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock -e HOME=/tmp --group-add 113'
                }
            }
            steps {
                script {
                    def tag = env.CHANGE_ID ? "pr-${env.CHANGE_ID}-${env.BUILD_NUMBER}" : "master-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = tag
                }
                sh 'docker build -t calculator-app:${IMAGE_TAG} .'
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    args '-e HOME=/tmp'
                }
            }
            steps {
                sh '''
                    pip install --no-cache-dir -r requirements.txt
                    python -m unittest discover -s tests
                '''
            }
        }

        stage('Push to ECR') {
            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock -e HOME=/tmp --group-add 113 -u root:root'
                }
            }
            steps {
                sh '''
                    apk add --no-cache python3 py3-pip
                    pip install --no-cache-dir --break-system-packages awscli
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                    docker tag calculator-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                    echo "Pushed image: ${ECR_REPO}:${IMAGE_TAG}"
                '''
            }
        }
    }
}
