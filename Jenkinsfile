pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'pujosn/simple-web:1.1.0'
        KUBE_CONFIG = credentials('gcp-key')
        GIT_REPO = 'git@github.com:pujosn/web-html.git'
        GCP_PROJECT = 'project-akhir-453413'
        GKE_ZONE = 'asia-southeast2-a'
        CLUSTER_NAME = 'cluster-project'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'git@github.com:pujosn/web-html.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t pujosn/simple-web:1.1.0 .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/']) {
                    sh 'docker build -t pujosn/simple-web:1.1.0'
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    sh '''
                    echo $KUBE_CONFIG > /tmp/gcp-key.json
                    gcloud auth activate-service-account --key-file=/tmp/gcp-key.json
                    gcloud container clusters get-credentials 'cluster-project' --zone asia-southeast2-a --project 'project-akhir-453413'
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }

        // stage('Monitor with Prometheus & Grafana') {
        //     steps {
        //         script {
        //             sh '''
        //             kubectl apply -f k8s/monitoring/prometheus.yaml
        //             kubectl apply -f k8s/monitoring/grafana.yaml
        //             '''
        //         }
        //     }
        // }
    }

    post {
        success {
            echo 'Deployment Success!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
