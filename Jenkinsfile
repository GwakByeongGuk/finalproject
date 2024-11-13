pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'gwakbyeongguk'
        DOCKER_BUILD_TAG = "20241108.${env.BUILD_NUMBER}"
        DOCKER_TOKEN = credentials('docker') // Docker Hub 자격 증명
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
                sh '''
                docker build --platform linux/arm64 -t ${DOCKER_IMAGE_OWNER}/prj-frontend:latest ./frontend
                docker build --platform linux/arm64 -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG} ./frontend
                docker build --platform linux/arm64 -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG} ./admin-service
                docker build --platform linux/arm64 -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG} ./visitor-service
                '''
            }
        }


        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USR', passwordVariable: 'DOCKER_PWD')]) {
                    sh "echo $DOCKER_PWD | docker login -u $DOCKER_USR --password-stdin"
                }
            }
        }

        stage('Docker Image Pushing') {
            steps {
                sh '''
                docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:latest
                docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}
                '''
            }
        }

        stage('Update ArgoCD values.yaml with Image Tags') {
            steps {
                dir('finalprojectargocd') {
                    sh """
                    sed -i "/${DOCKER_IMAGE_OWNER}\\/prj-frontend/{n;s/tag: \\".*\\"/tag: \\"${DOCKER_BUILD_TAG}\\"/}" deploy-argocd/values.yaml
                    sed -i "/${DOCKER_IMAGE_OWNER}\\/prj-admin/{n;s/tag: \\".*\\"/tag: \\"${DOCKER_BUILD_TAG}\\"/}" deploy-argocd/values.yaml
                    sed -i "/${DOCKER_IMAGE_OWNER}\\/prj-visitor/{n;s/tag: \\".*\\"/tag: \\"${DOCKER_BUILD_TAG}\\"/}" deploy-argocd/values.yaml
                    """
                }
            }
        }

        stage('Commit Changes') {
            steps {
                dir('finalprojectargocd') {
                    script{
                        def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                        if (changes) {
                            sh '''
                            git config user.name "gwakbyeongguk"
                            git config user.email "gwakbyeongguk@jenkins.com"
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
                        git pull origin main
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
