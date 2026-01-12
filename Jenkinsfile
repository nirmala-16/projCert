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
                    ssh -o StrictHostKeyChecking=no ec2-user@172.31.39.215 "
                        sudo dnf install -y https://yum.puppet.com/puppet8-release-el-9.noarch.rpm &&
                        sudo dnf install -y puppet-agent &&
                        sudo systemctl enable puppet &&
                        sudo systemctl start puppet
                    "
                    '''
                }
            }
        }

        stage('Job 2: Install Docker via Ansible') {
        steps {
        sshagent(credentials: ['ec2-user']) {
            sh '''
            ansible-playbook \
              -i inventory.ini \
              -u ec2-user \
              install-docker.yml
            '''
            }
        }
    }


        stage('Job 3: Build and Deploy PHP Docker Container') {
            steps {
                sshagent(credentials: ['ec2-user']) {
                    sh '''
                    rm -rf php-app || true
                    git clone ${GIT_REPO} php-app

                    scp -o StrictHostKeyChecking=no -r php-app \
                      ${SLAVE_USER}@${SLAVE_HOST}:/home/${SLAVE_USER}/php-app

                    ssh -o StrictHostKeyChecking=no ${SLAVE_USER}@${SLAVE_HOST} "
                        cd /home/${SLAVE_USER}/php-app &&
                        docker build -t ${DOCKER_IMAGE} . &&
                        docker rm -f ${DOCKER_CONTAINER} || true &&
                        docker run -d -p 8080:80 --name ${DOCKER_CONTAINER} ${DOCKER_IMAGE}
                    "
                    '''
                }
            }
        }
    }
}
