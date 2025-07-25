pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-app"
        IMAGE_TAG = "flask-app:${env.BUILD_NUMBER}"
    }

    stages {
        stage('Install dependencies') {
            steps {
                sh 'pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Run unit tests') {
            steps {
                sh 'pytest --junitxml=test-results.xml'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $IMAGE_TAG ."
                }
            }
        }
    }

    post {
        always {
            junit 'test-results.xml'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
