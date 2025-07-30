pipeline {
  agent any

  parameters {
    string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Git branch to deploy')
    string(name: 'ENV', defaultValue: 'dev', description: 'Target Environment')
    string(name: 'DEPLOY_VERSION', defaultValue: 'v1.0.0.2', description: 'Version tag or label')
  }

  environment {
    // Use Jenkins credentials securely
    SSH_KEY = credentials('EC2_SSH_KEY_ID') // Use Jenkins -> Credentials -> Add -> SSH key
    SERVER_USER = credentials('EC2_USER')   // e.g., 'ubuntu'
    SERVER_IP   = credentials('EC2_IP')     // IP of the EC2 instance
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: "${params.BRANCH_NAME}", url: 'https://github.com/VinchuKrutika/MyRectAppJenkinsDeployment.git'
      }
    }

    stage('Build') {
      steps {
        sh '''
          echo "Installing dependencies..."
          npm install

          echo "Building the application..."
          npm run build
        '''
      }
    }

    stage('Deploy to EC2') {
      steps {
        echo "Deploying to ${params.ENV}..."

        // Copy build files to EC2 instance using SSH
        sh '''
          echo "$SSH_KEY" > key.pem
          chmod 400 key.pem

          echo "Uploading files..."
          scp -o StrictHostKeyChecking=no -i key.pem -r build/* ${SERVER_USER}@${SERVER_IP}:/var/www/${params.ENV}

          echo "Reloading NGINX..."
          ssh -o StrictHostKeyChecking=no -i key.pem ${SERVER_USER}@${SERVER_IP} << EOF
            sudo systemctl reload nginx
EOF

          rm key.pem
        '''
      }
    }

    stage('Post-Deploy Validation') {
      steps {
        echo "Deployment completed for ${params.ENV} using branch ${params.BRANCH_NAME}."
      }
    }
  }

  post {
    failure {
      echo "Deployment failed. Consider triggering a rollback."
      // You can add rollback logic here based on previous deployment versions
    }
  }
}
