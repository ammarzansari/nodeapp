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
                ssh -o StrictHostKeyChecking=no $USER@$HOST << EOF
                    mkdir -p /home/ubuntu/app
                    exit
                EOF

                scp -o StrictHostKeyChecking=no -r . $USER@$HOST:/home/ubuntu/app

                ssh -o StrictHostKeyChecking=no $USER@$HOST << EOF
                    cd /home/ubuntu/app
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
