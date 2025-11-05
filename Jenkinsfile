@Library('lab4') _

pipeline {
    agent { label 'worker-agent' }

    environment {
        DOCKER_IMAGE = "mohamedmotaz350/app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        DEPLOYMENT_FILE = "deployment.yaml"
        SERVICE_FILE = "service.yaml"
        DOCKER_CRED_ID = "DockerHub"
        K8S_TOKEN_ID = "serviceaccount-token"
        K8S_APISERVER_ID = "APIServer"
    }

    stages {
        stage('Run Unit Tests') {
            steps {
                unitTest()
            }
        }

        stage('Build App') {
            steps {
                buildApp()
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                buildAndPushImage(DOCKER_IMAGE, IMAGE_TAG, DOCKER_CRED_ID)
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                deployToK8s(DEPLOYMENT_FILE, SERVICE_FILE, "${DOCKER_IMAGE}:${IMAGE_TAG}", K8S_TOKEN_ID, K8S_APISERVER_ID)
            }
        }
    }

    post {
        always { echo "Pipeline finished" }
        success { echo "Pipeline succeeded" }
        failure { echo "Pipeline failed" }
    }
}
