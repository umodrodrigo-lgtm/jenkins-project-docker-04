pipeline {
  agent any

  environment {
    APP_NAME  = "jenkins-flask-app"
    VENV_DIR  = ".venv"
    DOCKER_CREDS_ID = "docker-creds"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup') {
      steps {
        sh '''
          set -e
          python3 -m venv "${VENV_DIR}"
          . "${VENV_DIR}/bin/activate"
          python -m pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          set -e
          . "${VENV_DIR}/bin/activate"
          python -m pytest -q
        '''
      }
    }

    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_TOKEN')]) {
          sh '''
            set -e
            echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
          '''
        }
      }
    }

stage('Build Image') {
  steps {
    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_TOKEN')]) {
      sh '''
        set -e
        IMAGE_NAME="$DOCKERHUB_USER/${APP_NAME}"

        echo "Building: ${IMAGE_NAME}:latest"
        docker build -t "${IMAGE_NAME}:latest" .

        docker image ls | head -n 20
      '''
    }
  }
}

stage('Push Image') {
  steps {
    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_TOKEN')]) {
      sh '''
        set -e
        IMAGE_NAME="$DOCKERHUB_USER/${APP_NAME}"

        echo "Pushing: ${IMAGE_NAME}:latest"
        docker push "${IMAGE_NAME}:latest"
      '''
    }
  }
}

  post {
    always {
      sh 'docker logout || true'
      sh 'rm -rf .venv || true'
    }
  }
}