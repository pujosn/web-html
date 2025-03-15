pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pujosn/web-app"
        DOCKER_TAG = "1.1.2"
        GKE_CLUSTER = "cluster-development"
        GCP_PROJECT = "sanji-453509"
        DEPLOYMENT_NAME = "simple-app-9090"
        CONTAINER_NAME = "simple-app"
        zone = "asia-southeast2-a"
        SERVICE_PORT = "9090"
        EXTERNAL_IP = "34.101.133.48"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'git@github.com:pujosn/web-html.git'
            }
        }
        // update IP publik

        stage('Build Docker Image and Test') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker run --rm ${DOCKER_IMAGE}:${DOCKER_TAG} /bin/sh -c echo 'Running Tests'"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) { 
                    sh '''
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud container clusters get-credentials cluster-development --zone asia-southeast2-a --project sanji-453509
                    kubectl apply -f deployment.yaml
                    '''
                }
            }          
        }
    }

            stage('Expose Service') {
                steps {
                    script {
                    sh "kubectl expose deployment ${DEPLOYMENT_NAME} --type=LoadBalancer --name=${DEPLOYMENT_NAME}-service --port=${SERVICE_PORT} --target-port=${SERVICE_PORT}"
                }
            } 
        }
    }    
}

    post {
       success {
           echo 'Deployment berhasil!'
       }
       failure {
           echo 'Deployment gagal!'
       }
    }


        