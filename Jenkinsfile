pipeline {
    agent none

    parameters {
        string(name: 'APP_NAME', defaultValue: 'flask-app', description: 'Application name')
        string(name: 'DOCKER_REPO', defaultValue: 'nicolashm/flask-app', description: 'Docker Hub repository')
        string(name: 'HELM_CHARTS_REPO', defaultValue: 'https://github.com/nicolasholanda/sample-helm-charts.git', description: 'Helm charts repository URL')
        string(name: 'HELM_CHARTS_DIR', defaultValue: 'sample-helm-charts', description: 'Helm charts directory name')
        string(name: 'DEPLOY_NAMESPACE', defaultValue: 'apps', description: 'Kubernetes namespace for deployment')
        string(name: 'PYTHON_VERSION', defaultValue: '3.11', description: 'Python version for testing')
    }

    environment {
        BUILD_TAG = "${params.DOCKER_REPO}:${env.BUILD_NUMBER}"
        LOCAL_TAG = "${params.APP_NAME}:${env.BUILD_NUMBER}"
        
        DOCKER_CREDS_ID = 'dockerhub-creds'
        KUBECONFIG_CREDS_ID = 'jenkins-kubeconfig'
        
        TEST_RESULTS_FILE = 'test-results.xml'
    }

    stages {
        stage('Install & Test') {
            agent {
                docker { image "python:${params.PYTHON_VERSION}" }
            }
            steps {
                sh 'pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
                sh "pytest --junitxml=${env.TEST_RESULTS_FILE}"
            }
            post {
                always {
                    junit env.TEST_RESULTS_FILE
                }
            }
        }
        
        stage('Build Docker Image') {
            agent any
            steps {
                sh "docker build -t ${env.LOCAL_TAG} ."
            }
        }
        
        stage('Push Docker Image') {
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker tag ${env.LOCAL_TAG} ${env.BUILD_TAG}"
                    sh "docker push ${env.BUILD_TAG}"
                }
            }
        }
        
        stage('Clone Helm Charts') {
            agent any
            steps {
                sh "rm -rf ${params.HELM_CHARTS_DIR}"
                sh "git clone ${params.HELM_CHARTS_REPO}"
            }
        }
        
        stage('Deploy') {
            agent any
            steps {
                withKubeConfig([credentialsId: env.KUBECONFIG_CREDS_ID]) {
                    sh """
                        helm upgrade --install ${params.APP_NAME} ${params.HELM_CHARTS_DIR}/${params.APP_NAME} \\
                          --set image.repository=${params.DOCKER_REPO} \\
                          --set image.tag=${env.BUILD_NUMBER} \\
                          --namespace ${params.DEPLOY_NAMESPACE} \\
                          --create-namespace
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Build ${env.BUILD_NUMBER} completed for ${params.APP_NAME}"
        }
        success {
            echo "Successfully deployed ${params.APP_NAME} to ${params.DEPLOY_NAMESPACE}"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
