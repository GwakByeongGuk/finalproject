pipeline {
    agent any

	environment{
		DOCKER_IMAGE_OWNER = 'gwakbyeongguk'
		DOCKER_IMAGE_TAG = 'latest'
		DOCKER_TOKEN = credentials('dockerhub')
	}

    stages {
        stage('Clone from SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/GwakByeongGuk/finalproject.git'
            }
        }
		stage('Docker Image Building') {
            steps {
                sh '''
                cd finalproject
				docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG} ./frontend
				docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG} ./admin-service
				docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG} ./visitor-service
				'''
            }
        }
		stage('Docker Login') {
            steps {
				docker.withRegistry('', 'dockerhub') {
					// Docker login will happen automatically via withRegistry
				}
            }
        }
		stage('Docker Image Pushing') {
            steps {
                sh '''
				docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}
				docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG}
				docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG}
				'''
            }
        }
        stage('Trigger ArgoCD') {
           steps {
               script {
                 sh '''
                 curl -k -X POST https://13.125.240.87:32228/api/webhook -H "Content-Type: application/json" -d '{"ref": "refs/heads/main"}'
                 '''
               }
           }
        }
		stage('Docker Logout') {
            steps {
                sh 'docker logout'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
