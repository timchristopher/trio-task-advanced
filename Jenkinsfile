pipeline {
    agent any
    
    environment {
        PASSWORD = credentials('password')
    }
    parameters {
        booleanParam(name: 'skip_deploy', defaultValue: false, description: 'Set to false to deploy the application')
    }

    stages {
        stage('Pre-build Cleanup') {
            steps {
                echo 'Clean-up'
            }
        }
        stage('Build and Push') {
            steps {
                echo 'Build & Push'
                sh 'docker-compose build'
                sh 'docker-compose push'
            }
        }
        stage('Local clean-up') {
            sh 'docker rmi gcr.io/lbg-mea-11/trio-db:v2'
            sh 'docker rmi gcr.io/lbg-mea-11/trio-app:v2'
        }
        stage('Deploy to Cluster') {
            when { expression { params.skip_deploy != true } }
            steps {
                echo 'Deploy to Cluster'
                sh '''
                sed -e 's,{{PASSWORD}},'${PASSWORD}',g;' mysql-credentials.yaml | kubectl apply -f -
                kubectl apply -f nginx-config.yaml
                kubectl apply -f db.yaml
                sleep 5
                kubectl apply -f trio-deployment.yaml
                kubectl apply -f nginx-pod.yaml
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
