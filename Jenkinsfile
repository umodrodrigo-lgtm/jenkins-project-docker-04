pipeline {
  agent any

  environment {
    IMAGE_NAME = 'imalshar/jenkins-flask-app'
    IMAGE_TAG  = "${IMAGE_NAME}:${env.GIT_COMMIT}"
    VENV_DIR   = ".venv"
  }

  stages {

    stage('Setup') {
      steps {
        sh '''
          set -e
          python3 -m venv ${VENV_DIR}
          . ${VENV_DIR}/bin/activate
          python -m pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          set -e
          . ${VENV_DIR}/bin/activate
          python -m pytest -q
        '''
      }
    }

    stage('Login to docker hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh 'echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin'
        }
        echo 'Login successfully'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${IMAGE_TAG} .'
        echo "Docker image build successfully"
        sh 'docker image ls'
      }
    }

    stage('Push Docker Image') {
      steps {
        sh 'docker push ${IMAGE_TAG}'
        echo "Docker image push successfully"
      }
    }
  }
}