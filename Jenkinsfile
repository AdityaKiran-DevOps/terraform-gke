pipeline {
    agent any

    tools {
        sonarQube 'SonarScanner'
    }

    environment {
        PROJECT_ID = "project-122fc6d2-7f22-490c-ab6"
        REGION = "us-central1"
        REPOSITORY = "devops-repo"
        IMAGE = "terraform-gke"
        TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/AdityaKiran-DevOps/terraform-gke.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${TAG} .
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet

                docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${TAG}
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                gcloud container clusters get-credentials devops-gke \
                    --zone us-central1-a

                kubectl set image deployment/terraform-gke \
                terraform-gke=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${TAG} \
                -n devops

                kubectl rollout status deployment/terraform-gke -n devops
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
