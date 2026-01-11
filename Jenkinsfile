pipeline {
    agent {
        label 'jslave'
    }

    stages {
        stage('Install and configure puppet agent') {
            steps {
                script {
                    // Job 1: Install and configure puppet agent
                    // Install Puppet Agent
                    sh 'curl -O https://apt.puppetlabs.com/puppet6-release-bionic.deb'
                    sh 'sudo dpkg -i puppet6-release-bionic.deb'
                    sh 'sudo apt-get update'
                    sh 'sudo apt-get install -y puppet-agent'

                    // Configure Puppet Agent
                    sh 'echo "server = puppetmaster.example.com" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf'
                    sh 'sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true'
                }
            }
        }

        stage('Push Ansible configuration to install Docker') {
            steps {
                script {
                    //Job 2: Push Ansible configuration on the test server to install Docker
                    sh "rm -rf /tmp/ansibletemp ||:"
                    sh "git clone https://github.com/Khaganshu-RK/Edureka-DevOps.git /tmp/ansibletemp"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@ip-172-31-24-180 'rm -rf ~/installation.yml ||:'"
                    sh "exit"
                    sh "scp /tmp/ansibletemp/installation.yml ubuntu@ip-172-31-24-180:~/"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@ip-172-31-24-180 'ansible-playbook -v ~/installation.yml'"
                    sh "exit"
                    sh "rm -rf /tmp/ansibletemp"
                }
            }
        }

        stage('Build and deploy PHP Docker container with try catch block') {
            steps {
                // Job 3: Pull PHP website and Dockerfile from Git repo, build, and deploy container
                script {
                    try {
                        sh "rm -rf /tmp/php-web ||:"
                        sh "git clone https://github.com/Khaganshu-RK/Edureka-DevOps.git /tmp/php-web"
                        sh "cd /tmp/php-web"
                        sh 'docker build -t php-website .'
                        sh 'docker run -d -p 8010:80 php-website'
                    } catch (Exception e) {
                        sh 'docker rm -f $(docker ps -aq)'
                        sh 'docker image prune -a --force'
                        echo "Deployment failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
        
//        stage('Build and deploy PHP Docker container') {
//            steps {
//                // Job 3: Pull PHP website and Dockerfile from Git repo, build, and deploy container
//                sh "rm -rf /tmp/php-web ||:"
//                sh "git clone https://github.com/Khaganshu-RK/Edureka-DevOps.git /tmp/php-web"
//                sh "cd /tmp/php-web"
//                sh 'docker build -t php-website .'
//                sh 'docker run -d -p 8010:80 php-website'
//            }
//        }
//
//        stage('Cleanup on failure') {
//            steps {
//                // Job 4: Delete the running container on the test server if Job 3 fails
//                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
//                    sh 'docker container prune --force --filter "until=3m"'
//                    sh 'docker image prune -a --force --filter "until=5m"'
//                }
//            }
//        }
    }    
}
