pipeline {
    agent none

    environment {
        IMAGE_NAME = "flask-app"
        IMAGE_TAG = "flask-app:${env.BUILD_NUMBER}"
        CHARTS_REPO = "https://github.com/nicolasholanda/sample-helm-charts.git"
        CHARTS_DIR = "sample-helm-charts"
        DEPLOY_NAMESPACE = "apps"
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
                sh 'docker build -t $IMAGE_TAG .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE_REPO:$IMAGE_TAG'
                }
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
                    sh '''
                        helm upgrade --install flask-app $CHARTS_DIR/flask-app \
                          --set image.repository=$IMAGE_NAME \
                          --set image.tag=$BUILD_NUMBER \
                          --namespace $DEPLOY_NAMESPACE \
                          --create-namespace
                    '''
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
