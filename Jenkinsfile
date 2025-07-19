pipeline {
  agent any

  environment {
    NODE_IMAGE = 'node:18-alpine'
    ARTIFACT_NAME = 'app.zip'
    NEXUS_URL = 'http://18.208.214.3:8081'
    NEXUS_REPO = 'my-node-repo'
    NEXUS_CRED_ID = 'nexus-creds'
    DEPLOY_SERVER = 'ec2-user@18.208.214.3'
    DEPLOY_DIR = '/usr/share/nginx/html'
    DEPLOY_KEY_ID = 'nginx-ssh-creds'
  }

stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build in Docker') {
      steps {
        script {
          docker.image("${NODE_IMAGE}").inside('-v $PWD:/app -w /app') {
            sh 'npm install'
            sh 'npm run build'
            sh 'zip -r ${ARTIFACT_NAME} dist || zip -r ${ARTIFACT_NAME} build'
          }
        }
      }
    }


    stage('Upload to Nexus') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${NEXUS_CRED_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh '''
            curl -v -u $USERNAME:$PASSWORD --upload-file ${ARTIFACT_NAME} ${NEXUS_URL}/repository/${NEXUS_REPO}/${ARTIFACT_NAME}
          '''
        }
      }
    }

    stage('Deploy to Nginx Server') {
      steps {
        sshagent (credentials: ["${DEPLOY_KEY_ID}"]) {
          sh """
            scp ${ARTIFACT_NAME} ${DEPLOY_SERVER}:/tmp/
            ssh ${DEPLOY_SERVER} 'unzip -o /tmp/${ARTIFACT_NAME} -d ${DEPLOY_DIR} && rm /tmp/${ARTIFACT_NAME}'
          """
        }
      }
    }
  }

  post {
    success {
      echo '✅ Node.js build, Nexus upload, and deployment succeeded.'
    }
    failure {
      echo '❌ Something went wrong during pipeline execution.'
    }
  }
}
