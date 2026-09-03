pipeline {
    agent any

    parameters {
        choice(name: 'DEPLOY_BRANCH', choices: ['main', 'dev'], description: 'Select environment/branch to deploy')
    }

    tools {
        nodejs 'Node_7.8.0'
    }

    environment {
        DOCKER_IMAGE   = "node${params.DEPLOY_BRANCH}"
        IMAGE_TAG      = "v1.0"
        CONTAINER_NAME = "app-${params.DEPLOY_BRANCH}"
        APP_PORT       = "${params.DEPLOY_BRANCH == 'main' ? '3000' : '3001'}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.DEPLOY_BRANCH}",
                    url: 'https://github.com/Harshithaxx18/cicd-pipeline.git',
                    credentialsId: 'github-creds'
            }
        }
        stage('Build') {
            steps { sh 'npm install' }
        }
        stage('Test') {
            steps { sh 'npm test || echo "no tests defined"' }
        }
        stage('Build Docker Image') {
            steps { sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ." }
        }
        stage('Deploy') {
            steps {
                sh """
                    if [ \$(docker ps -aq -f name=^${CONTAINER_NAME}\$) ]; then
                        docker rm -f ${CONTAINER_NAME}
                    fi
                """
                sh "docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:3000 ${DOCKER_IMAGE}:${IMAGE_TAG}"
            }
        }
    }
}
