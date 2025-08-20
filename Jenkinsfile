pipeline {
    agent any

    tools {
        nodejs "Node 18"
    }

    environment {
        IMAGE_MAIN = "nodemain:v1.0"
        IMAGE_DEV  = "nodedev:v1.0"
        DOCKER_CREDENTIALS = 'dockerhub_credential'
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }


    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", credentialsId: 'github-https-creds', url: 'https://github.com/Rauulhub/cicd-pipeline-jenkins.git'
            }
        }
        stage('Check Docker') {
            steps {
                sh 'which docker'
                sh 'docker --version'
            }
        }
        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == "main") {
                        sh "docker build -t ${IMAGE_MAIN} ."
                    } else if (env.BRANCH_NAME == "dev") {
                        sh "docker build -t ${IMAGE_DEV} ."
                    }
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub_credential') {
                    def app = docker.image("rauulhub/nodemain:v1.0")
                    app.push()
            }
            }
        }
        }
    }
}
