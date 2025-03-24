def remote = [:]
def git_url = "git@github.com:chesnokov70/node-app-multy-ec2.git"
def instances = ['ec2-instance-1', 'ec2-instance-2', 'ec2-instance-3', 'ec2-instance-4', 'ec2-instance-5']

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
      stage('Configure credentials') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh_instance_key', 
                                                   keyFileVariable: 'private_key', 
                                                   usernameVariable: 'username')]) {
                    script {
                        instances.each { instance ->
                            node(instance) {
                                remote.name = "${env.HOST}"
                                remote.host = "${env.HOST}"
                                remote.user = "$username"
                                remote.identity = readFile("$private_key")
                                remote.allowAnyHosts = true
                            }
                        }
                    }
                }
            }
        }
      stage('Clone repo') {
            steps {
                script {
                    instances.each { instance ->
                        node(instance) {
                            withCredentials([sshUserPrivateKey(credentialsId: 'ssh_github_access_key', 
                                                               keyFileVariable: 'GIT_SSH_KEY')]) {
                                sh 'mkdir -p ~/.ssh'
                                sh 'cp $GIT_SSH_KEY ~/.ssh/id_rsa'
                                sh 'chmod 600 ~/.ssh/id_rsa'
                                sh 'ssh-keyscan github.com >> ~/.ssh/known_hosts'
                            }

                            checkout([
                                $class: 'GitSCM', 
                                branches: [[name: "${revision}"]], 
                                doGenerateSubmoduleConfigurations: false, 
                                extensions: [], 
                                submoduleCfg: [], 
                                userRemoteConfigs: [[credentialsId: 'ssh_github_access_key', url: "$git_url"]]
                            ])
                        }
                    }
                }
            }
        }

      stage('Build and push') {
            steps {
                script {
                    instances.each { instance ->
                          node(instance) {
                              sh """ 
                              docker login -u chesnokov70 -p $TOKEN
                              docker build -t "${env.REGISTRY}:${env.BUILD_ID}" .
                              docker push "${env.REGISTRY}:${env.BUILD_ID}"
                              """
                          }
                    }
                }
            }        
      }           
  }    
} 
