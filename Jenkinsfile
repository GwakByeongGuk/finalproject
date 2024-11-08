pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'gwakbyeongguk'
        DOCKER_BUILD_TAG = "20241108.${env.BUILD_NUMBER}"
        DOCKER_TOKEN = credentials('dockerhub') // Docker Hub 자격 증명
        GIT_CREDENTIALS = credentials('github')
        REPO_URL = 'GwakByeongGuk/finalproject.git'
        COMMIT_MESSAGE = 'Update README.md via Jenkins Pipeline'
    }

    stages {
        stage('Clone from SCM') {
            steps {
                sh'''
                    rm -rf project-jenkins finalproject
                    git branch: 'main', url: 'https://github.com/GwakByeongGuk/finalproject.git'
                    git branch: 'main', url: 'https://github.com/GwakByeongGuk/finalprojectargo.git'
                '''
            }
        }

        stage('Docker Image Building') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:lastet ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG} ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG} ./admin-service"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG} ./admin-service"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG} ./visitor-service"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG} ./visitor-service"
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
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:latest"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-admin:latest"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:latest"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}"
                }
            }
        }
        
        stage('Update ArgoCD Deployment YAML with Image Tags') {
            steps {
                sh '''
                sed -i 's|image: {{.Values.image.admin.repository}}:{{.Values.image.admin.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                sed -i 's|image: {{.Values.image.visitor.repository}}:{{.Values.image.visitor.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                sed -i 's|image: {{.Values.image.frontend.repository}}:{{.Values.image.frontend.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                '''
            }
        }
        
        stage('Commit Changes') {
            steps {
                dir('finalproject') {
                    sh '''
                    git config user.name "GwakByeongGuk"
                    git config user.email "GwakByeongGuk@jenkins.com"
                    git add deploy-argocd/templates/deployment.yaml
                    git commit -m "Update image tags to ${DOCKER_IMAGE_TAG}"
                    git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${REPO_URL} main
                    '''
                }
            }
        }

        stage('Push Changes') {
            steps {
                script {
                    sh '''
                    git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/GwakByeongGuk/finalproject.git main
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
