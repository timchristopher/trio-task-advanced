pipeline {
    agent any
    
    environment {
        PASSWORD = credentials('password')
    }
    parameters {
        booleanParam(name: 'stopService', defaultValue: false, description: 'Set to true to stop the application')
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
            steps {
                sh 'docker rmi gcr.io/lbg-mea-11/trio-db:v2'
                sh 'docker rmi gcr.io/lbg-mea-11/trio-app:v2'
            }
        }
        stage('Deploy to Cluster') {
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
        stage('Wait for External IP') {
            steps {
                sh '''#!/bin/bash
                kubectl get svc
                bash -c 'external_ip=""; while [ -z $external_ip ]; do echo "Waiting for end point..."; external_ip=$(kubectl get svc nginx --template="{{range .status.loadBalancer.ingress}}{{.ip}}{{end}}"); [ -z "$external_ip" ] && sleep 2; done; echo "End point ready-" && echo $external_ip; export endpoint=$external_ip'
                kubectl describe svc nginx
                '''
            }
        }
        stage('Stop the service') {
            when { expression { params.stopService == true } }
            steps {
                echo 'Stop the service'
                sh '''
                kubectl delete -f nginx-pod.yaml
                kubectl delete -f trio-deployment.yaml
                kubectl delete -f db.yaml
                kubectl delete -f nginx-config.yaml
                sed -e 's,{{PASSWORD}},'${PASSWORD}',g;' mysql-credentials.yaml | kubectl delete -f -
                '''
            }
        }
    }

}
