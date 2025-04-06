pipeline {
        
    agent {label 'kaniko'}

    options {
        disableConcurrentBuilds()
    }

    environment {
        AWS_REGION = 'us-east-1'
        ACCOUNT_ID = '234382811800'
        REPO_NAME = 'my-app'
        TERRAFORM_ECS_MANIFESTS_REPO = 'https://github.com/ahmedKhaled1995/kaniko-ecs-fargate-manifests-demo.git'
        TERRAFORM_ECS_MANIFESTS_BRANCH = 'my-app'
        TERRAFORM_ECS_MANIFESTS_CLONE_FOLDER_PATH = "ecs"
    }

    stages {

        stage('Setting-up variables') {
            steps {
                script{
                    env.TAG=sh(
                        returnStdout: true,
                            script: '''#!/bin/bash
                                COMMIT_COUNT=$(git rev-list --count HEAD)
                                SHORT_HASH=$(git rev-parse --short HEAD)
                                echo ${COMMIT_COUNT}.${SHORT_HASH}
                            '''
                    ).trim()
                    echo "The tag is ${env.TAG}"
                    if (env.GIT_BRANCH == 'develop') {
                        env.TERRAFORM_WORKSPACE = "dev"
                        env.TERRAFORM_WORKSPACE_TFVARS = "dev.tfvars"
                        env.TAG = "${env.TAG}-dev"
                    } else if (env.GIT_BRANCH == 'main'){
                        env.TERRAFORM_WORKSPACE = "prod"
                        env.TERRAFORM_WORKSPACE_TFVARS = "prod.tfvars"
                        env.TAG = "${env.TAG}-prod"
                    } else{
                        currentBuild.result = 'FAILURE'
                        error("Branch is not configured")
                    }
                }
            }
        }

        // stage('Build Docker Image') {
        //     steps {
        //             sh '''
        //                 /kaniko/executor \
        //                 --context ${WORKSPACE} \
        //                 --dockerfile ${WORKSPACE}/Dockerfile \
        //                 --destination ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${TAG}
        //             '''
        //     }
        // }

        stage('Pulling ecs Deployment Files') {
            steps {
                dir("${TERRAFORM_ECS_MANIFESTS_CLONE_FOLDER_PATH}") {
                    git branch: "${TERRAFORM_ECS_MANIFESTS_BRANCH}", credentialsId: 'git_repo_creds', url: "${TERRAFORM_ECS_MANIFESTS_REPO}"
                }

            }
        }

        // terraform apply -var-file=./${TERRAFORM_WORKSPACE_TFVARS} --auto-approve
        stage("Deploy ECS") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AWS_CRED', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    echo "Deploying ECS..."
                    sh '''
                        export REPO="${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
                        sed -i.bak "s|IMAGE_VALUE|${REPO}|g" ./${TERRAFORM_ECS_MANIFESTS_CLONE_FOLDER_PATH}/${TERRAFORM_WORKSPACE_TFVARS}
                        sed -i.bak "s|TAG_VALUE|${TAG}|g" ./${TERRAFORM_ECS_MANIFESTS_CLONE_FOLDER_PATH}/${TERRAFORM_WORKSPACE_TFVARS}

                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}

                        cd ${TERRAFORM_ECS_MANIFESTS_CLONE_FOLDER_PATH}
                        
                        terraform init

                        # Check if the workspace exists, create it if it doesn’t
                        if terraform workspace list | grep -q "^\\s*${TERRAFORM_WORKSPACE}$"; then
                            echo "Workspace '${TERRAFORM_WORKSPACE}' exists. Selecting it..."
                            terraform workspace select "${TERRAFORM_WORKSPACE}"
                        else
                            echo "Workspace '${TERRAFORM_WORKSPACE}' does not exist. Creating it..."
                            terraform workspace new "${TERRAFORM_WORKSPACE}"
                        fi

                        terraform plan -var-file=./${TERRAFORM_WORKSPACE_TFVARS}
                    '''
                }
            }
        }


    }

    post {
        cleanup{
            cleanWs()
        }        
    }
}