pipeline {
    agent { label 'jslave' }

    environment {
        SLAVE_HOST = "172.31.39.215"
        SLAVE_USER = "ec2-user"
        GIT_REPO   = "https://github.com/nirmala-16/projCert.git"
        DOCKER_IMAGE = "my-php-webapp"
        DOCKER_CONTAINER = "php-web"
    }

    stages {

        stage('Job 1: Install Puppet Agent on Remote') {
            steps {
                sshagent(credentials: ['ec2-user']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${SLAVE_USER}@${SLAVE_HOST} << EOF
                      sudo yum install -y puppet-agent || sudo apt-get install -y puppet-agent
                      sudo systemctl enable puppet
                      sudo systemctl start puppet
                    EOF
                    '''
                }
            }
        }

        stage('Job 2: Install Docker via Ansible') {
            steps {
                sh '''
                ansible-playbook -i inventory.ini install-docker.yml
                '''
            }
        }

        stage('Job 3: Build and Deploy PHP Docker Container') {
            steps {
                sshagent(credentials: ['ec2-user']) {
                    sh '''
                    rm -rf php-app || true
                    git clone ${GIT_REPO} php-app

                    scp -o StrictHostKeyChecking=no -r php-app ${SLAVE_USER}@${SLAVE_HOST}:/home/${SLAVE_USER}/php-app

                    ssh -o StrictHostKeyChecking=no ${SLAVE_USER}@${SLAVE_HOST} << EOF
                      cd /home/${SLAVE_USER}/php-app
                      docker build -t ${DOCKER_IMAGE} .
                      docker rm -f ${DOCKER_CONTAINER} || true
                      docker run -d -p 8080:80 --name ${DOCKER_CONTAINER} ${DOCKER_IMAGE}
                    EOF
                    '''
                }
            }
        }
    }
}
