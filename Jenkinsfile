pipeline {
    agent none

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "992382545251.dkr.ecr.us-east-1.amazonaws.com/stav-calculator-app"
        IMAGE_TAG  = "pr-${env.CHANGE_ID}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build Image') {
            agent {
                docker {
                    image 'docker:24-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker build -t calculator-app:${IMAGE_TAG} .'
            }
        }
    }
}
