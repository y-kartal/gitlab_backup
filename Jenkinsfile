pipeline {
    agent any
    stages {
        stage('Deploy GitLab Docker Container') {
            steps {
                sh '''
                    mkdir srv && cd srv && mkdir gitlab && cd gitlab
                    export GITLAB_HOME=/srv/gitlab
                    docker run --detach \
                        --hostname localhost \
                        --publish 443:443 --publish 80:80 \
                        --name gitlab \
                        --restart always \
                        --volume gitlab-config:/etc/gitlab \
                        --volume gitlab-logs:/var/log/gitlab \
                        --volume gitlab-data:/var/opt/gitlab \
                        gitlab/gitlab-ce:16.8.2-ce.0
                '''
            }
        }
        stage('List Docker Images') {
            steps {
                sh 'docker image ls'
            }
        }
        
        stage('List Docker Containers') {
            steps {
                sh 'docker container ls'
            }
        }

        stage("Wait Prior Starting Smoke Testing") {
            steps {
                script {
                    def time = 200 
                    echo "Waiting ${time} seconds for deployment to complete prior to starting smoke testing"
                    sleep time // seconds
                }
            }
        }

        stage('Stop Puma') {
            steps {
                sh 'docker exec gitlab gitlab-ctl stop puma'
            }
        }
        
        stage('Stop Sidekiq') {
            steps {
                sh 'docker exec gitlab gitlab-ctl stop sidekiq'
            }
        }

        stage('Copy Backup Files to GitLab Container') {
            steps {
                //sh 'docker cp /home/kupkup16122012/1717498171_2024_06_04_16.8.2_gitlab_backup.tar gitlab:/var/opt/gitlab/backups'
                sh 'docker cp /home/kupkup16122012/gitlab.rb gitlab:/etc/gitlab'
                sh 'docker cp /home/kupkup16122012/gitlab-secrets.json gitlab:/etc/gitlab'
            }
        }

        stage('Restore Backup') {
            steps {
                sh 'docker exec gitlab gitlab-backup restore BACKUP=/home/kupkup16122012/1717498171_2024_06_04_16.8.2'
            }
        }

        stage('Restart GitLab') {
            steps {
                sh 'docker restart gitlab'
            }
        }

        stage('Check GitLab Configuration') {
            steps {
                sh 'docker exec gitlab gitlab-rake gitlab:check SANITIZE=true'
            }
        }
    }

    post {
        failure {
            echo 'Deleting the docker container due to pipeline failure'
            sh 'docker rm -f $(docker ps -aq)'
        }
    }
}
