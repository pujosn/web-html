pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pujosn/simple-web"
        DOCKER_TAG = "1.1.0"
        GKE_CLUSTER = "cluster-development"
        GCP_PROJECT = "sanji-453509"
        DEPLOYMENT_NAME = "simple-web-deployment"
        CONTAINER_NAME = "simple-web"
        zone = "asia-southeast2-a"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'git@github.com:pujosn/web-html.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
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

    post {
       success {
           echo 'Deployment berhasil!'
       }
       failure {
           echo 'Deployment gagal!'
       }
    }
  }

}        