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
    }
}
