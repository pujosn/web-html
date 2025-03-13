pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'pujosn/simple-web:1.1.0'
        KUBE_CONFIG = credentials('gcp-key')
        GIT_REPO = 'git@github.com:pujosn/web-html.git'
        GCP_PROJECT = 'project-akhir-453413'
        GKE_ZONE = 'asia-southeast2-a'
        CLUSTER_NAME = 'cluster-development'
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
                    sh 'docker build -t pujosn/simple-web:1.1.0 .'
                }
            }
        }

        // stage('push gke') {
        //     steps {
        //         withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
        //             sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
        //     }
        // }

        //     }

        stage('Deploy to GKE') {
             steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    echo "Checking Google Credentials: $GOOGLE_APPLICATION_CREDENTIALS"
                    ls -l $GOOGLE_APPLICATION_CREDENTIALS
                    cp $GOOGLE_APPLICATION_CREDENTIALS /tmp/gcp-key.json
                    chmod 600 /tmp/gcp-key.json
                    gcloud auth activate-service-account --key-file=/tmp/gcp-key.json
                    export USE_GKE_GCLOUD_AUTH_PLUGIN=True
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
