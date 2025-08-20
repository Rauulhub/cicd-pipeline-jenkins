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
        stage('Stop old containers') {
            steps {
                script {
                    if (env.BRANCH_NAME == "main") {
                        sh "docker ps -q --filter ancestor=${IMAGE_MAIN} | xargs -r docker stop"
                    } else if (env.BRANCH_NAME == "dev") {
                        sh "docker ps -q --filter ancestor=${IMAGE_DEV} | xargs -r docker stop"
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    if (env.BRANCH_NAME == "main") {
                        sh "/usr/local/bin/docker run -d --expose 3000 -p 3000:3000 ${IMAGE_MAIN}"
                    } else if (env.BRANCH_NAME == "dev") {
                        sh "/usr/local/bin/docker run -d --expose 3001 -p 3001:3000 ${IMAGE_DEV}"
                    }
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    sh """
                        echo $env.DOCKER_PASSWORD | /usr/local/bin/docker login -u $env.DOCKER_USERNAME --password-stdin
                    """
                    if (env.BRANCH_NAME == "main") {
                        sh """
                            /usr/local/bin/docker tag ${IMAGE_MAIN} rauulhub/${IMAGE_MAIN}
                            /usr/local/bin/docker push rauulhub/${IMAGE_MAIN}
                        """
                    } else if (env.BRANCH_NAME == "dev") {
                        sh """
                            /usr/local/bin/docker tag ${IMAGE_DEV} rauulhub/${IMAGE_DEV}
                            /usr/local/bin/docker push rauulhub/${IMAGE_DEV}
                        """
                    }
                }
            }
        }
        
    }
}
