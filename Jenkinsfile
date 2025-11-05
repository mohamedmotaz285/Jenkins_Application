pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "mohamedmotaz350/app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        DEPLOYMENT_FILE = "deployment.yaml"
        SERVICE_FILE = "service.yaml"
    }

    stages {

        stage('Run Unit Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Build App') {
            steps {
                sh "mvn package "
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DockerHub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    docker login -u "$DOCKER_USER" -p "$DOCKER_PASS"
                    docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Delete Local Image') {
            steps {
                sh "docker rmi ${DOCKER_IMAGE}:${IMAGE_TAG} || true"
            }
        }

        stage('Update Deployment File') {
            steps {
                sh """
                sed -i 's|image:.*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|g' ${DEPLOYMENT_FILE}
                """
            }
        }

        stage('Deploy to Kubernetes (ServiceAccount)') {
            steps {
                withCredentials([
                    string(credentialsId: 'APIServer', variable: 'APISERVER'),
                    string(credentialsId: 'serviceaccount-token', variable: 'TOKEN')
                ]) {
                    sh """
                    kubectl apply \
                        --server="$APISERVER" \
                        --token="$TOKEN" \
                        --insecure-skip-tls-verify=true \
                        -f ${DEPLOYMENT_FILE}

                    kubectl apply \
                        --server="$APISERVER" \
                        --token="$TOKEN" \
                        --insecure-skip-tls-verify=true \
                        -f ${SERVICE_FILE}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished "
        }
        success {
            echo "Pipeline succeeded "
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
