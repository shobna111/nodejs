pipeline {
  agent any

  environment {
    NODE_IMAGE = 'node:18-alpine'
    ARTIFACT_NAME = 'app.zip'
    NEXUS_URL = 'http://18.208.214.3:8081'
    NEXUS_REPO = 'my-node-repo'
    NEXUS_CRED_ID = 'nexus-creds'
    DEPLOY_SERVER = 'user@18.208.214.3'
    DEPLOY_DIR = '/var/www/html/app'
    DEPLOY_KEY_ID = 'nginx-ssh-creds'
  }

  stages {
    stage('Cleanup') {
      steps {
        sh 'rm -rf node_modules bower_components dist build *.zip'
      }
    }

    stage('Build in Docker') {
      steps {
        script {
          docker.image("${NODE_IMAGE}").inside {
            sh '''
              npm install
              npm install -g bower
              bower install --allow-root
              npm run build || echo "No build script found, skipping..."
              zip -r ${ARTIFACT_NAME} . -x "*.git*" "node_modules/*"
            '''
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
