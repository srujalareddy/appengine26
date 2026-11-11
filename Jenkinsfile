// Jenkinsfile for a Jenkins VM that has a service account attached.
// No key file or Jenkins credential is needed: gcloud uses the VM's
// service account automatically (metadata server).

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
                // Keep .gcloudignore so venv/ is not uploaded
                sh 'rm -rf staging/'
            }
        }

        stage('Verify GCP Access') {
            steps {
                echo "===== Verifying GCP access (VM service account) ====="
                sh '''
                    echo "Active account:"
                    gcloud auth list
                    gcloud config set project "$PROJECT_ID"

                    gcloud auth print-access-token > /dev/null
                    echo "GCP authentication successful"
                '''
            }
        }

        stage('Deploy to App Engine') {
            steps {
                echo "===== Deploying App Engine ====="
                sh '''
                    gcloud app deploy app.yaml \
                      --project="$PROJECT_ID" \
                      --quiet \
                      --verbosity=info
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
