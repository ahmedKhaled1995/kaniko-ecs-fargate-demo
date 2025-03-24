pipeline {
    agent {
            label 'kaniko'
        }
    environment {
        AWS_REGION = 'us-east-1'
        ACCOUNT_ID = '234382811800'
        REPO_NAME = 'kaniko-demo'
    }
    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/ahmedKhaled1995/kaniko-ecs-fargate-demo.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                        --context ${WORKSPACE} \
                        --dockerfile ${WORKSPACE}/Dockerfile \
                        --destination ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:latest
                    '''
                }
            }
        }
    }
}
