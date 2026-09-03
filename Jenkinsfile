pipeline {
  agent any

  tools {
    nodejs 'Node_7.8.0'
  }

  environment {
    APP_NAME   = "nodeapp"
    VERSION    = "v1.0"
    BRANCH     = "${env.BRANCH_NAME}"
    ENV_NAME   = "${env.BRANCH_NAME == 'main' ? 'main' : 'dev'}"
    PORT       = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    IMAGE_NAME = "node${env.BRANCH_NAME == 'main' ? 'main' : 'dev'}:${VERSION}"
    CONT_NAME  = "node-${env.BRANCH_NAME == 'main' ? 'main' : 'dev'}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'echo "Branch: $BRANCH, ENV: $ENV_NAME, PORT: $PORT"'
      }
    }

    stage('Build') {
      steps {
        sh 'npm install'
        sh 'npm run build || true'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test || true'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          docker build -t ${IMAGE_NAME} .
          docker images | head
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          echo "Deploying ${IMAGE_NAME} on port ${PORT} with container ${CONT_NAME}"

          # advanced task: remove only the container for this env
          if docker ps -a --format '{{.Names}}' | grep -w ${CONT_NAME}; then
            docker rm -f ${CONT_NAME}
          fi

          docker run -d --name ${CONT_NAME} -p ${PORT}:3000 ${IMAGE_NAME}

          docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
        '''
      }
    }
  }
}
