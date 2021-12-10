pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Branch name to build.')
        string(name: 'REPO_URL', defaultValue: 'git@bitbucket.org:besafx-bitbucket/food-cloud.git', description: 'Repository GIT URL.')
        string(name: 'CREDENTIALS_ID', defaultValue: 'microzilla-git-id', description: 'Credentials id stored on Jenkins.')
        booleanParam(name: 'INIT_JASPER_SERVER', description: "Reinitialize Jasper Server from scratch.", defaultValue: false)
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: params.BRANCH_NAME, credentialsId: params.CREDENTIALS_ID, url: params.REPO_URL
            }
        }
        stage('Initialize') {
            steps {
                echo "Creating root directory for images at /usr/share/microzilla"
                sh "sudo mkdir -p /usr/share/microzilla"
                echo "Creating user defined network used by all containers"
                sh 'sudo docker network create microzilla_network || true'
            }
        }
        stage('Jasper Server') {
            steps{
                script {
                    if (params.INIT_JASPER_SERVER == true) {

                        echo "Deleting volume of MariaDB if exist"
                        sh 'sudo docker rm -f microzilla_mariadb_data || true'

                        echo "Creating volume for MariaDB"
                        sh 'sudo docker volume create --name microzilla_mariadb_data'

                        echo "Deleting MariaDB Container if exist"
                        sh 'sudo docker rm -f microzilla_mariadb || true'

                        echo "Creating MariaDB Container"
                        sh """\
                           sudo docker run -d --name microzilla_mariadb \
                           -e ALLOW_EMPTY_PASSWORD=yes \
                           -e MARIADB_USER=cloud \
                           -e MARIADB_DATABASE=cloud \
                           --net microzilla_network \
                           --restart always \
                           --volume microzilla_mariadb_data:/bitnami \
                           bitnami/mariadb:latest
                           """

                        echo "Deleting volume of Jasper Server if exist"
                        sh 'sudo docker rm -f microzilla_jasper_data ||true'

                        echo "Creating volume for Jasper Server"
                        sh 'sudo docker volume create --name microzilla_jasper_data'

                        echo "Deleting Jasper Server Container if exist"
                        sh 'sudo docker rm -f microzilla_jasper || true'

                        echo "Creating Jasper Server Container"
                        sh """\
                           sudo docker run -d --name microzilla_jasper \
                           -p 8500:8080 \
                           -e ALLOW_EMPTY_PASSWORD=yes \
                           -e MARIADB_HOST=microzilla_mariadb \
                           -e JASPERREPORTS_DATABASE_USER=cloud \
                           -e JASPERREPORTS_DATABASE_NAME=cloud \
                           -e JASPERREPORTS_USERNAME=cloud \
                           -e JASPERREPORTS_PASSWORD=cloud \
                           -e JASPERREPORTS_EMAIL=cloud@cloud.com \
                           --net microzilla_network \
                           --restart always \
                           --volume microzilla_jasper_data:/bitnami \
                           bitnami/jasperreports:latest
                           """

                    } else {
                        echo 'Skipping initializing Jasper Server.'
                    }
                }
            }
        }
    }
}
