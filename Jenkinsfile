pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }
    stages {
        stage('Setup') {
            steps {
                sh '''
                bash -c 'source myenv/bin/activate'
                myenv/bin/pip install -r requirements.txt
                '''
            }
        }
        stage('Test') {
            steps {
                sh "pytest"
            }
        }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
                sh "ls -lart"
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip  ${username}@${SERVER_IP}:/home/ubuntu/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/ubuntu/myapp.zip -d /home/ubuntu/app/
                        source /home/ubuntu/app/myenv/bin/activate
                        cd /home/ubuntu/app/
                        pip install -r requirements.txt
                        sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
       
        
       
        
    }
}
