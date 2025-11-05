@Library('lab4') _

pipeline {
    agent { label 'worker-agent' }

    environment {
        DOCKER_IMAGE = "mohamedmotaz350/app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        DEPLOYMENT_FILE = "deployment.yaml"
        SERVICE_FILE = "service.yaml"
    }

    stages {
        stage('Run Unit Test') {
            steps { unitTest() }
        }

        stage('Build App') {
            steps { buildApp() }
        }

        stage('Build & Push Docker Image') {
            steps { buildAndPushImage(DOCKER_IMAGE, 'DockerHub') }
        }

        stage('Deploy to Kubernetes') {
            steps { deployToK8s(DEPLOYMENT_FILE, SERVICE_FILE, 'APIServer', 'serviceaccount-token') }
        }
    }

    post {
        always { echo "Pipeline finished" }
        success { echo "Pipeline succeeded" }
        failure { echo "Pipeline failed" }
    }
}
