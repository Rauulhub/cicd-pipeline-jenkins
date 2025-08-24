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
                sh "which docker"
                sh "docker --version"
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
                        sh "docker ps -a -q --filter ancestor=${IMAGE_MAIN} | xargs -r docker rm"
                    } else if (env.BRANCH_NAME == "dev") {
                        // Mata cualquier contenedor usando el puerto 3001
                        sh """
                        CONTAINER_ID=\$(docker ps -q --filter "publish=3001")
                        if [ ! -z "\$CONTAINER_ID" ]; then
                          docker stop \$CONTAINER_ID
                          docker rm -f \$CONTAINER_ID
                        fi
                        """
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
                       // docker.withRegistry('', DOCKER_CREDENTIALS) {
                       //if (env.BRANCH_NAME == "main") {
                       //    sh "docker tag ${IMAGE_MAIN} rauulhub/${IMAGE_MAIN}"
                       //     sh "docker push rauulhub/${IMAGE_MAIN}"
                       // } else if (env.BRANCH_NAME == "dev") {
                       //     sh "/usr/local/bin/docker login -u ${env.DOCKER_USER} -p ${env.DOCKER_PASS}"
                       //     sh "docker tag ${IMAGE_DEV} rauulhub/${IMAGE_DEV}"
                       //     sh "docker push rauulhub/${IMAGE_DEV}"
                    withCredentials([
                    string(credentialsId: 'dockerhub-username', variable: 'DOCKER_USER'),
                    string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASS')
                     ]) {
                        sh "/usr/local/bin/docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                
                        if (env.BRANCH_NAME == "main") {
                         sh "/usr/local/bin/docker tag ${IMAGE_MAIN} rauulhub/${IMAGE_MAIN}"
                         sh "/usr/local/bin/docker push rauulhub/${IMAGE_MAIN}"
                        } else if (env.BRANCH_NAME == "dev") {
                         sh "/usr/local/bin/docker tag ${IMAGE_DEV} rauulhub/${IMAGE_DEV}"
                         sh "/usr/local/bin/docker push rauulhub/${IMAGE_DEV}"
                         }
                      }
                    }
                }
        
        }
    }
}
      

