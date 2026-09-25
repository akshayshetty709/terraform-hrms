pipeline {
    agent any
 
    stages {
 
        stage('Deploy to EC2') {
            steps {
                sshagent(['hrms-key']) {
                    sh '''
                        ssh -A \
                          -o StrictHostKeyChecking=no \
                          -o ServerAliveInterval=30 \
                          -o ServerAliveCountMax=20 \
                          -o ConnectTimeout=30 \
                          ubuntu@3.109.36.186 "
                            set -e
                            
                            rm -rf /home/ubuntu/Backend
 
                           
                            git clone -b main git@github.com:HRMsOwn/Backend.git /home/ubuntu/Backend
 
                            cd /home/ubuntu/Backend
 
                           
                            docker compose down || true
                            docker system prune -f
 
                            
                            docker compose up -d
            
                        "
                    '''
                }
            }
        }
    }
}
