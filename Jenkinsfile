pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'gwakbyeongguk'
        DOCKER_IMAGE_TAG = 'latest'
        DOCKER_TOKEN = credentials('dockerhub') // Docker Hub 자격 증명
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
                    // 각 서비스에 대해 Docker 이미지 빌드
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG} ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG} ./admin-service"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG} ./visitor-service"
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    // Docker Hub에 로그인
                    sh "echo ${DOCKER_TOKEN_PSW} | docker login -u ${DOCKER_TOKEN_USR} --password-stdin"
                }
            }
        }

        stage('Docker Image Pushing') {
            steps {
                script {
                    // Docker Hub에 Docker 이미지 푸시
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Trigger ArgoCD') {
            steps {
                script {
                    // ArgoCD 트리거
                    sh '''
                    curl -k -X POST https://13.125.240.87:32228/api/webhook -H "Content-Type: application/json" -d '{"ref": "main"}'
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
