pipeline {
    agent any

    environment {
        HOST = "10.0.154.49"
        USER = "ubuntu"
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Deploy to Private EC2') {
    steps {
        sshagent(['node.pem']) {
            sh '''
            # Create tarball
            tar czf app.tar.gz app.js package.json ecosystem.config.js

            # Ensure target directory
            ssh -o StrictHostKeyChecking=no $USER@$HOST "mkdir -p /home/ubuntu/app"

            # Copy tarball
            scp -o StrictHostKeyChecking=no app.tar.gz $USER@$HOST:/home/ubuntu/app

            # Deploy on EC2 with PM2 home set
            ssh -o StrictHostKeyChecking=no $USER@$HOST << 'EOF'
                export PM2_HOME=/home/ubuntu/.pm2
                cd /home/ubuntu/app
                tar xzf app.tar.gz
                npm install
                pm2 delete node-app || true
                pm2 start ecosystem.config.js
                pm2 save
EOF
                    '''
                }
            }
        }
    }
}
