pipeline {
    agent none

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "992382545251.dkr.ecr.us-east-1.amazonaws.com/stav-calculator-app"
        PRODUCTION_IP = "34.201.132.77"
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
                    def tag = env.CHANGE_ID ? "pr-${env.CHANGE_ID}-${env.BUILD_NUMBER}" : env.GIT_COMMIT.take(7)
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
                    image 'python:3.11-slim'
                    args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker -e HOME=/tmp --group-add 113 -u root:root'
                }
            }
            steps {
                sh '''
                    set -e
                    export PATH=$PATH:/root/.local/bin
                    pip install --no-cache-dir --quiet awscli
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                    docker tag calculator-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                    echo "Pushed image: ${ECR_REPO}:${IMAGE_TAG}"
                '''
            }
        }

        stage('Deploy to Production') {
            when { branch 'master' }
            agent any
            steps {
                sshagent(credentials: ['production-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@${PRODUCTION_IP} "
                            aws ecr get-login-password --region ${AWS_REGION} | sudo docker login --username AWS --password-stdin ${ECR_REPO} &&
                            sudo docker pull ${ECR_REPO}:${IMAGE_TAG} &&
                            sudo docker stop calculator-app || true &&
                            sudo docker rm calculator-app || true &&
                            sudo docker run -d --name calculator-app -p 5000:5000 ${ECR_REPO}:${IMAGE_TAG}
                        "
                    '''
                }
            }
        }

        stage('Health Check') {
            when { branch 'master' }
            agent any
            steps {
                sh '''
                    for i in 1 2 3 4 5; do
                        if curl -sf http://${PRODUCTION_IP}:5000/health; then
                            echo "Health check passed"
                            exit 0
                        fi
                        echo "Attempt $i failed, retrying in 5s..."
                        sleep 5
                    done
                    echo "Health check failed after 5 attempts"
                    exit 1
                '''
            }
        }
    }
}
