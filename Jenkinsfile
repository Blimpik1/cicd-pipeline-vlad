pipeline {
  agent any

  tools {
    nodejs 'node-7.8.0'
  }

  environment {
    APP_NAME = "nodeapp"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'node -v && npm -v'
      }
    }

    stage('Build') {
      steps {
        sh 'npm install'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          def branch = env.BRANCH_NAME ?: 'main'
          if (branch == 'main') {
            env.IMAGE_NAME = 'nodemain:v1.0'
            env.HOST_PORT  = '3000'
            env.CONTAINER  = 'node-main'
          } else if (branch == 'dev') {
            env.IMAGE_NAME = 'nodedev:v1.0'
            env.HOST_PORT  = '3001'
            env.CONTAINER  = 'node-dev'
          } else {
            error("Unsupported branch: ${branch}")
          }

          sh "docker build -t ${env.IMAGE_NAME} ."
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          // Stop/remove only the container for this env (lower downtime, doesn't kill other env)
          sh """
            if docker ps -a --format '{{.Names}}' | grep -qx '${env.CONTAINER}'; then
              docker rm -f '${env.CONTAINER}'
            fi
          """

          // App inside container listens on 3000 (Node app), we map host port depending on env
          sh "docker run -d --name ${env.CONTAINER} --expose 3000 -p ${env.HOST_PORT}:3000 ${env.IMAGE_NAME}"
        }
      }
    }
  }
}
