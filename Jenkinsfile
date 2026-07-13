pipeline {
    agent any

    environment {
        PROJECT_ID = "project-122fc6d2-7f22-490c-ab6"
        REGION = "us-central1"
        REPOSITORY = "devops-repo"
        IMAGE = "terraform-gke"
        CLUSTER = "devops-gke"
        ZONE = "us-central1-a"
        NAMESPACE = "devops"
        TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                docker --version
                kubectl version --client
                terraform version
                gcloud --version
                '''
            }
        }

        stage('Configure GCP') {
            steps {
                sh '''
                gcloud config set project ${PROJECT_ID}
                gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build \
                -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${TAG} .
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push \
                ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${TAG}
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                gcloud container clusters get-credentials ${CLUSTER} \
                --zone ${ZONE}

                kubectl set image deployment/${IMAGE} \
                ${IMAGE}=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${TAG} \
                -n ${NAMESPACE}

                kubectl rollout status deployment/${IMAGE} \
                -n ${NAMESPACE}
                '''
            }
        }
    }

    post {

        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
