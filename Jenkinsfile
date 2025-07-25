pipeline {
    agent none

    environment {
        IMAGE_NAME = "flask-app"
        IMAGE_TAG = "flask-app:${env.BUILD_NUMBER}"
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
                script {
                    sh "docker build -t $IMAGE_TAG ."
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
