pipeline {
  agent any

  environment {
    IMAGE_NAME = 'imalshar/jenkins-flask-app'   // change to YOUR docker hub username/repo
    VENV_DIR   = ".venv"
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

    stage('Login to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh 'echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin'
        }
        echo 'Login successfully'
      }
    }

    stage('Build Image') {
      steps {
        sh '''
          set -e
          docker build -t "${IMAGE_NAME}:latest" .
          docker image ls | head -n 20
        '''
        echo "Docker image build successfully"
      }
    }

    stage('Push Image') {
      steps {
        sh '''
          set -e
          docker push "${IMAGE_NAME}:latest"
        '''
        echo "Docker image push successfully"
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