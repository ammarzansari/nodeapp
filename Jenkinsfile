pipeline {
    agent any

    environment {
        HOST = "10.0.154.49"
        USER = "ubuntu"
        PM2_HOME = "/home/ubuntu/.pm2"
    }

    stages {

        stage('Install Dependencies Locally') {
            steps {
                sh 'npm install'
            }
        }

        stage('Load .env from Jenkins Secret') {
            steps {
                withCredentials([file(credentialsId: 'buildspec', variable: 'ENV_FILE')]) {
                    sh '''
                        cp "$ENV_FILE" .env

                        echo "===== .env loaded successfully ====="
                        # ⚠️ Debug only (remove in production)
                        cat .env
                    '''
                }
            }
        }

        stage('Package App') {
            steps {
                sh '''
                    # ✅ Include .env in package
                    tar czf app.tar.gz app.js package.json ecosystem.config.js .env
                '''
            }
        }

        stage('Deploy to Private EC2') {
            steps {
                sshagent(['node.pem']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $USER@$HOST "mkdir -p /home/ubuntu/app"
                        scp -o StrictHostKeyChecking=no app.tar.gz $USER@$HOST:/home/ubuntu/app/

                        ssh -o StrictHostKeyChecking=no $USER@$HOST << 'EOF'
                            export PM2_HOME=/home/ubuntu/.pm2
                            mkdir -p $PM2_HOME
                            chown -R ubuntu:ubuntu $PM2_HOME

                            cd /home/ubuntu/app

                            tar xzf app.tar.gz

                            # Verify .env exists
                            if [ -f .env ]; then
                                echo ".env file deployed successfully"
                            else
                                echo ".env file missing!"
                                exit 1
                            fi

                            npm install

                            pm2 delete node-app || true
                            pm2 start ecosystem.config.js
                            pm2 save

                            # ❌ pm2 startup removed; run manually once
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
