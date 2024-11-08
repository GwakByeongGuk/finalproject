pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'gwakbyeongguk'
        DOCKER_IMAGE_TAG = 'latest'
        DOCKER_BUILD_TAG = 'v${env.BUILD_NUMBER}'
        DOCKER_TOKEN = credentials('dockerhub') // Docker Hub 자격 증명
        GIT_CREDENTIALS = credentials('github')
        REPO_URL = 'GwakByeongGuk/finalproject.git'
        COMMIT_MESSAGE = 'Update README.md via Jenkins Pipeline'
    }

    stages {
        stage('Clone from SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/GwakByeongGuk/finalproject.git'
            }
        }

        stage('Docker Image Building') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG} ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG} ./admin-service"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG} ./visitor-service"
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    sh "echo ${DOCKER_TOKEN_PSW} | docker login -u ${DOCKER_TOKEN_USR} --password-stdin"
                }
            }
        }

        stage('Docker Image Pushing') {
            steps {
                script {
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Trigger ArgoCD') {
            steps {
                script {
                    sh '''
                    curl -k -X POST https://15.164.179.57:31380/api/webhook -H "Content-Type: application/json" -d '{"ref": "main"}'
                    '''
                }
            }
        }
        
        stage('Commit Changes') {
            steps {
                dir('finalproject') {
                    sh '''
                    git config user.name "admin"
                    git config user.email "admin@jenkins.com"
                    git add README.md
                    git commit -m "${COMMIT_MESSAGE}"
                    '''
                }
            }
        }

        stage('Push Changes') {
            steps {
                dir('finalproject') {
                    sh '''
                    git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${REPO_URL} main
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
