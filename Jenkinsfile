pipeline {
    agent none

    environment {
        IMAGE_NAME = "flask-app"
        IMAGE_TAG = "flask-app:${env.BUILD_NUMBER}"
        CHARTS_REPO = "https://github.com/nicolasholanda/sample-helm-charts.git"
        CHARTS_DIR = "sample-helm-charts"
    }

    stages {
        stage('Install & Test') {
            agent {
                docker { image 'python:3.11' }
            }
            steps {
                sh 'pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
                sh 'pytest --junitxml=test-results.xml'
            }
            post {
                always {
                    junit 'test-results.xml'
                }
            }
        }
        stage('Build Docker Image') {
            agent any
            steps {
                sh 'eval $(minikube docker-env) && docker build -t flask-app:latest .'
            }
        }
        stage('Clone Helm Charts') {
            agent any
            steps {
                sh 'rm -rf $CHARTS_DIR'
                sh 'git clone $CHARTS_REPO'
            }
        }
        stage('Deploy') {
            agent any
            steps {
                withKubeConfig([credentialsId: 'jenkins-kubeconfig']) {
                    sh 'helm upgrade --install flask-app $CHARTS_DIR/flask-app --set image.repository=flask-app --set image.tag=latest'
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
