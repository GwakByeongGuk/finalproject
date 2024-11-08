pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'gwakbyeongguk'
        DOCKER_BUILD_TAG = "20241108.${env.BUILD_NUMBER}"
        DOCKER_TOKEN = credentials('dockerhub') // Docker Hub 자격 증명
        GIT_CREDENTIALS = credentials('github')
        REPO_URL = 'GwakByeongGuk/finalproject.git'
        ARGOCD_REPO_URL = 'GwakByeongGuk/finalprojectargocd.git'
        COMMIT_MESSAGE = 'Update README.md via Jenkins Pipeline'
    }

    stages {
        stage('Clone from SCM') {
            steps {
                dir('finalproject') {
                    git branch: 'main', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${REPO_URL}"
                }
                dir('project-argocd') {
                    git branch: 'main', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL}"
                }
            }
        }

        stage('Docker Image Building') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:latest ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG} ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG} ./admin-service"
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
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}"
                }
            }
        }

        stage('Update ArgoCD Deployment YAML with Image Tags') {
            steps {
                dir('finalprojectargocd') {
                    sh """
                    sed -i "s|image: {{.Values.image.admin.repository}}:{{.Values.image.admin.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}|g" finalprojectargocd/templates/deployment.yaml
                    sed -i "s|image: {{.Values.image.visitor.repository}}:{{.Values.image.visitor.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}|g" finalprojectargocd/templates/deployment.yaml
                    sed -i "s|image: {{.Values.image.frontend.repository}}:{{.Values.image.frontend.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}|g" finalprojectargocd/templates/deployment.yaml
                    """
                }
            }
        }
        
        stage('Commit Changes') {
            steps {
                dir('finalprojectargocd') {
                    script {
                        def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                        if (changes) {
                            sh '''
                            git config user.name "GwakByeongGuk"
                            git config user.email "GwakByeongGuk@jenkins.com"
                            git add deploy-argocd/templates/deployment.yaml
                            git commit -m "Update image tags to ${DOCKER_BUILD_TAG}"
                            '''
                        } else {
                            echo "No changes to commit"
                        }
                    }
                }
            }
        }

        stage('Push Changes') {
            steps {
                dir('finalprojectargocd') {
                    script {
                        sh "git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL} main"
                    }
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
