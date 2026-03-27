pipeline {
agent any

```
environment {
    HOST = "10.0.154.49"
    USER = "ubuntu"
}

stages {

    stage('Clone Repo') {
        steps {
            git 'https://github.com/ammarzansari/nodeapp.git'
        }
    }

    stage('Install Dependencies') {
        steps {
            sh 'npm install'
        }
    }

    stage('Deploy to Private EC2') {
        steps {
            sshagent(['node']) {
                sh '''
                ssh -o StrictHostKeyChecking=no $USER@$HOST << EOF
                    cd /home/newuser || mkdir -p /home/newuser
                    exit
                EOF

                scp -o StrictHostKeyChecking=no -r * $USER@$HOST:/home/ubuntu/app

                ssh -o StrictHostKeyChecking=no $USER@$HOST << EOF
                    cd /home/newuser
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
```

}
