pipeline {
    agent any

    environment {
        PROJECT_ID = "project-122fc6d2-7f22-490c-ab6"
        REGION = "us-central1"
        REPOSITORY = "devops-repo"
        IMAGE = "terraform-gke"
        TAG = "v${BUILD_NUMBER}"

        IMAGE_NAME = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${TAG}"
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
                script {

                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {

                        withCredentials([
                            string(
                                credentialsId: 'sonarqube-token',
                                variable: 'SONAR_TOKEN'
                            )
                        ]) {

                            sh """
                                ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=terraform-gke \
                                -Dsonar.projectName=terraform-gke \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=$SONAR_HOST_URL \
                                -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {

                sh """
                    gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet

                    docker push ${IMAGE_NAME}
                """
            }
        }

        stage('Deploy to GKE') {

            steps {

                sh """

                    gcloud container clusters get-credentials devops-gke \
                    --zone us-central1-a

                    kubectl set image deployment/terraform-gke \
                    terraform-gke=${IMAGE_NAME} \
                    -n devops

                    kubectl rollout status deployment/terraform-gke \
                    -n devops

                """
            }
        }

        stage('Verify Deployment') {

            steps {

                sh '''

                kubectl get deployment -n devops

                kubectl get pods -n devops

                kubectl get svc -n devops

                '''

            }

        }

    }

    post {

        success {

            echo "====================================="
            echo "Pipeline Completed Successfully"
            echo "Docker Image : ${IMAGE_NAME}"
            echo "Application Deployed Successfully"
            echo "====================================="

        }

        failure {

            echo "====================================="
            echo "Pipeline Failed"
            echo "Check Console Output"
            echo "====================================="

        }

        always {

            cleanWs()

        }

    }
}
