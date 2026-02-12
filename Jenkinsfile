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
            IMAGE_TAG_COMMIT="${IMAGE_NAME}:${GIT_COMMIT}"
            IMAGE_TAG_LATEST="${IMAGE_NAME}:latest"

            echo "Building: $IMAGE_TAG_COMMIT"
            docker build -t "$IMAGE_TAG_COMMIT" .

            echo "Tagging: $IMAGE_TAG_LATEST"
            docker tag "$IMAGE_TAG_COMMIT" "$IMAGE_TAG_LATEST"

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
            IMAGE_TAG_COMMIT="${IMAGE_NAME}:${GIT_COMMIT}"
            IMAGE_TAG_LATEST="${IMAGE_NAME}:latest"

            echo "Pushing: $IMAGE_TAG_COMMIT"
            docker push "$IMAGE_TAG_COMMIT"

            echo "Pushing: $IMAGE_TAG_LATEST"
            docker push "$IMAGE_TAG_LATEST"
          '''
        }
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