pipeline {
    agent any

    environment {
        HOST = "10.0.154.49"   // Private EC2
        USER = "ubuntu"
        PM2_HOME = "/home/ubuntu/.pm2"
    }

    stages {

        stage('Install Dependencies Locally') {
            steps {
                sh 'npm install'
            }
        }

        stage('Package App') {
            steps {
                sh '''
                # Create tarball of only necessary files
                tar czf app.tar.gz app.js package.json ecosystem.config.js
                '''
            }
        }

        stage('Deploy to Private EC2') {
            steps {
                sshagent(['node.pem']) {
                    sh '''
                    # Ensure deployment folder exists
                    ssh -o StrictHostKeyChecking=no $USER@$HOST "mkdir -p /home/ubuntu/app"

                    # Copy tarball
                    scp -o StrictHostKeyChecking=no app.tar.gz $USER@$HOST:/home/ubuntu/app

                    # Deploy on EC2
                    ssh -o StrictHostKeyChecking=no $USER@$HOST << 'EOF'
                        # Set PM2 home to avoid permission issues
                        export PM2_HOME=/home/ubuntu/.pm2
                        mkdir -p $PM2_HOME
                        chown -R ubuntu:ubuntu $PM2_HOME

                        cd /home/ubuntu/app

                        # Extract new app files
                        tar xzf app.tar.gz

                        # Install Node dependencies
                        npm install

                        # Stop old PM2 process if exists
                        pm2 delete node-app || true

                        # Start new app via PM2
                        pm2 start ecosystem.config.js

                        # Save process list
                        pm2 save

                        # Configure PM2 to auto-start on reboot
                        pm2 startup systemd -u ubuntu --hp /home/ubuntu
EOF
                    '''
                }
            }
        }

        stage('Clean Up Local Tarball') {
            steps {
                sh 'rm -f app.tar.gz'
            }
        }
    }
}
