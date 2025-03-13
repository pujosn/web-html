pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'pujosn/simple-web:1.1.0'
        KUBE_CONFIG = credentials('kubeconfig')
        GIT_REPO = 'git@github.com:pujosn/web-html.git'
        GCP_PROJECT = 'project-akhir-453413'
        GKE_ZONE = 'asia-southeast2-a'
        CLUSTER_NAME = 'cluster-development'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: "${GITHUB_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Membangun image Docker
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login ke Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', 'pujosn/simple-web') { // Ganti dengan credential Docker Hub
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Mengkonfigurasi kubectl menggunakan kubeconfig
                    writeFile(file: 'kubeconfig', text: "${KUBE_CONFIG}")
                    sh 'gcloud auth activate-service-account --key-file=kubeconfig'
                    sh "gcloud container clusters get-credentials ${GKE_CLUSTER} --zone ${GKE_ZONE}"
                    
                    // Melakukan deploy ke Kubernetes
                    sh "kubectl set image deployment/web-deployment web=${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed. Please check the logs.'
        }
    }
}