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
                dir('finalprojectargocd') {
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

        stage('Update ArgoCD values.yaml with Image Tags') {
            steps {
                dir('finalprojectargocd') {
                    sh """
                    sed -i "s|frontend:.*|frontend:\\n    repository: ${DOCKER_IMAGE_OWNER}/prj-frontend\\n    tag: \\\"${DOCKER_BUILD_TAG}\\\"|g" deploy-argocd/values.yaml
                    sed -i "s|admin:.*|admin:\\n    repository: ${DOCKER_IMAGE_OWNER}/prj-admin\\n    tag: \\\"${DOCKER_BUILD_TAG}\\\"|g" deploy-argocd/values.yaml
                    sed -i "s|visitor:.*|visitor:\\n    repository: ${DOCKER_IMAGE_OWNER}/prj-visitor\\n    tag: \\\"${DOCKER_BUILD_TAG}\\\"|g" deploy-argocd/values.yaml
                    """
                }
            }
        }

        
        stage('Commit Changes') {
            steps {
                dir('finalprojectargocd') {
                    script {
                        def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                        echo "Changes detected: ${changes}"
                        if (changes) {
                            sh '''
                            git config user.name "GwakByeongGuk"
                            git config user.email "GwakByeongGuk@jenkins.com"
                            git add deploy-argocd/values.yaml
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
                    withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_CREDENTIALS_USR', passwordVariable: 'GIT_CREDENTIALS_PSW')]) {
                        sh '''
                        git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL} main
                        '''
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
