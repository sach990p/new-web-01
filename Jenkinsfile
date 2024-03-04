pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'AZURE_CREDENTIALS'
        SSH_PRIVATE_KEY_ID = 'SSH_PRIVATE_KEY_ID'
        DOCKER_HUB_CREDENTIALS_ID = 'DOCKER_HUB_CREDENTIALS_ID'
        GIT_CREDENTIALS_ID = 'GIT_CREDENTIALS_ID'
    }
    stages {
        stage('Provision VM with Terraform') {
            steps {
                script {
                    // Execute Terraform init and apply
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }
        stage('Install Docker and Dependencies') {
            steps {
                sshagent(credentials: [env.SSH_PRIVATE_KEY_ID]) {
                    // SSH into the newly created VM and install Docker
                    sh 'ssh -o StrictHostKeyChecking=no adminuser@$(terraform output vm_ip) "bash -s" < install_docker.sh'
                }
            }
        }
        stage('Deploy Application') {
            steps {
                sshagent(credentials: [env.SSH_PRIVATE_KEY_ID]) {
                    // SSH into VM, pull code, build Docker image, push to Docker Hub, and run
                    sh 'ssh -o StrictHostKeyChecking=no adminuser@$(terraform output vm_ip) "bash -s" < fetch_and_build.sh'
                    sh 'ssh -o StrictHostKeyChecking=no adminuser@$(terraform output vm_ip) "bash -s" < run_container.sh'
                }
            }
        }
    }
}
