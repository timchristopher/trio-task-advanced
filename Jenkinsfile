pipeline {
    agent any
   
    stages {
        stage('Clone Repo') {
            steps {
                echo 'Not required as git clone is handled by Jenkins directly'
                //git url: 'https://github.com/timchristopher/duo-task', branch: 'master'
            }
        }
        stage('Pre-build Cleanup') {
            steps {
                echo 'Clean-up'
                // sh 'docker system prune -f'

                sh '''#!/bin/bash
                declare -a arr=("mysql" "flask-app" "trio-nginx")
                for i in "${arr[@]}"
                do
                    echo "Clean-up: $i"
                    docker ps -q --filter "name=^$i\$" | grep -q . && docker stop $i | (echo -n "Stopped " && cat) || echo "$i not running"
                    docker ps -qa --filter "name=^$i\$" | grep -q . && docker rm $i | (echo -n "Removed container " && cat) || echo "Container named '$i' does not exist"
                    echo
                done
                unset arr
                
                docker network ls -q --filter "name=^trio-net\$" | grep -q . && docker network rm trio-net || echo "trio-net does not exist"
                '''
            }
        }
        stage('Build') {
            steps {
                echo 'Build'
                sh '''
                cd db
                docker build -t trio-db:v1 .
                cd ..
                
                cd flask-app
                docker build -t trio-app:v1 .
                cd ..
                '''
            }
        }
        stage('Run') {
            steps {
                echo 'Run'
                sh '''
                docker network ls -q --filter "name=^trio-net\$" | grep -q . && echo "trio-net already exists" || docker network create trio-net
                
                docker run -d --network trio-net --name mysql trio-db:v1
                docker run -d --network trio-net --name flask-app trio-app:v1
                docker run -d --network trio-net --name trio-nginx -p 80:80 --mount type=bind,source=$(pwd)/nginx/nginx.conf,target=/etc/nginx/nginx.conf  nginx:alpine
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Test'
            }
        }
    }

}
