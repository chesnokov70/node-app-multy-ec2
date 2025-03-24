def remote = [:]
def git_url = "git@github.com:chesnokov70/node-app-multy-ec2.git"
pipeline {
  agent any
  parameters {
    gitParameter (name: 'revision', type: 'PT_BRANCH')
  }
  environment {
    REGISTRY = "chesnokov70/node-app-multy-ec2"
    SSH_KEY_PATH = '/home/jenkins/.ssh/id_ed25519.pub'
    SSH_KEY = credentials('ssh_instance_key')
    TOKEN = credentials('hub_token')
    AWS_REGION = 'us-east-1'
  }
  stages {
      stage('Checkout Code') {
            steps {
                git 'git@github.com:chesnokov70/node-app-multy-ec2.git'
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                sh '''
                cd terraform
                terraform init
                terraform plan -out=tfplan
                '''
            }
        }

        stage('Terraform Apply') {
            steps {
                sh '''
                cd terraform
                terraform apply -auto-approve
                '''
            }
        }

        stage('Copy SSH Key to Instances') {
            steps {
                script {
                    def instances = sh(
                        script: '''
                        aws ec2 describe-instances \
                            --query "Reservations[*].Instances[*].PublicIpAddress" \
                            --filters "Name=tag:Name,Values=your-ec2-tag" \
                            --region $AWS_REGION \
                            --output text
                        ''',
                        returnStdout: true
                    ).trim().split()

                    instances.each { ip ->
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${ip} \
                        "sudo mkdir -p /root/.ssh && \
                        sudo chmod 700 /root/.ssh && \
                        echo '${readFile(SSH_KEY_PATH)}' | sudo tee -a /root/.ssh/authorized_keys && \
                        sudo chmod 600 /root/.ssh/authorized_keys"
                        """
                    }
                }
            }
        }

        stage('Deploy App') {
            steps {
                script {
                    def instances = sh(
                        script: '''
                        aws ec2 describe-instances \
                            --query "Reservations[*].Instances[*].PublicIpAddress" \
                            --filters "Name=tag:Name,Values=your-ec2-tag" \
                            --region $AWS_REGION \
                            --output text
                        ''',
                        returnStdout: true
                    ).trim().split()

                    instances.each { ip ->
                        sh """
                        ssh -o StrictHostKeyChecking=no root@${ip} << EOF
                        sudo apt-get update
                        sudo apt-get install -y docker.io
                        docker run -d -p 8080:8080 your-app-image:latest
                        EOF
                        """
                    }
                }
            }
        }
   
  }    
} 
