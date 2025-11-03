pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/owusurk'  // <------ change this only if needed
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/Owusurk/225-lab3-3.git'  // <------ change to this repo
        KUBECONFIG = credentials('owusurk-225')  // <------ your K8s credentials
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint index.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_CREDENTIALS_ID}") {
                        docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    sh "sed -i 's|cithit/owusurk:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|g' deployment-dev.yaml"
                    sh 'kubectl apply -f deployment-dev.yaml'
                }
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                sh 'kubectl get all'
            }
        }
    }

    post {
        success {
            slackSend teamDomain: 'cit225-fall25', channel: '#builds', color: 'good', tokenCredentialId: 'slack-225', message: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        failure {
            slackSend teamDomain: 'cit225-fall25', channel: '#builds', color: 'danger', tokenCredentialId: 'slack-225', message: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
