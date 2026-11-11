pipeline {
    agent any

    environment {
        PROJECT_ID = 'project-00867faf-aa1c-482c-a67'
    }

    stages {

        stage('Setup Python') {
            steps {
                echo "===== Setting up Python ====="
                sh '''
                    python3 --version
                    rm -rf venv
                    python3 -m venv venv

                    . venv/bin/activate
                    python --version
                    pip install --upgrade pip setuptools wheel
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Clean Workspace') {
            steps {
                echo "===== Cleaning old deployment artifacts ====="
                // Do NOT delete .gcloudignore: it keeps venv/ out of the upload
                sh 'rm -rf staging/'
            }
        }

        stage('Authenticate GCP') {
            steps {
                echo "===== Authenticating GCP ====="
                withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GCP_KEY_FILE')]) {
                    sh '''
                        echo "Activating service account..."
                        gcloud auth activate-service-account --key-file="$GCP_KEY_FILE"
                        gcloud config set project "$PROJECT_ID"

                        echo "Checking GCP authentication..."
                        gcloud auth print-access-token > /dev/null
                        echo "GCP authentication successful"
                    '''
                }
            }
        }

        stage('Deploy to App Engine') {
            steps {
                echo "===== Deploying App Engine ====="
                sh '''
                    gcloud app deploy app.yaml --quiet --verbosity=info
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo "Deployment succeeded!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
