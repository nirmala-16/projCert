pipeline {
    agent { label 'jslave' }

    environment {
        SLAVE_HOST = "172.31.39.215"          // Replace with your slave/test server hostname or IP
        SLAVE_USER = "ec2-user"            // Replace with your EC2 username (ec2-user or ubuntu)
        GIT_REPO   = "https://github.com/nirmala-16/projCert.git"
        DOCKER_IMAGE = "my-php-webapp"
        DOCKER_CONTAINER = "php-web"
    }

    stages {
        stage('Job 1: Install Puppet Agent on Slave') {
            steps {
                script {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${SLAVE_USER}@${SLAVE_HOST} '
                        sudo yum install -y puppet-agent || sudo apt-get install -y puppet-agent
                        sudo systemctl enable puppet
                        sudo systemctl start puppet
                    '
                    """
                }
            }
        }

        stage('Job 2: Install Docker via Ansible') {
            steps {
                script {
                    // Assuming you have inventory.ini and playbook install-docker.yml in Jenkins workspace
                    sh """
                    ansible-playbook -i inventory.ini install-docker.yml
                    """
                }
            }
        }

        stage('Job 3: Build and Deploy PHP Docker Container') {
            steps {
                script {
                    sh """
                    # Clone repo
                    rm -rf php-app || true
                    git clone ${GIT_REPO} php-app

                    # Build Docker image on slave
                    scp -r php-app ${SLAVE_USER}@${SLAVE_HOST}:/home/${SLAVE_USER}/php-app
                    ssh ${SLAVE_USER}@${SLAVE_HOST} '
                        cd php-app &&
                        docker build -t ${DOCKER_IMAGE} .
                        docker rm -f ${DOCKER_CONTAINER} || true
                        docker run -d -p 8080:80 --name ${DOCKER_CONTAINER} ${DOCKER_IMAGE}
                    '
                    """
                }
            }
        }
    }

    
}
