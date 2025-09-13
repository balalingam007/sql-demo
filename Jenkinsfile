pipeline {
    agent any

    environment {
        ARTIFACT_URL = "http://artifactory:8081/artifactory/generic-local/sql-demo/artifact.zip"
        GIT_REPO     = "https://github.com/balalingam007/sql-demo.git"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build') {
            steps {
                sh 'zip -r artifact.zip *.sql'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'artifact.zip', fingerprint: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner -Dsonar.projectKey=sql-demo -Dsonar.sources=.'
                }
            }
        }

        stage('Publish Artifact') {
            steps {
                sh """
                  curl -u admin:password -T artifact.zip \
                  http://artifactory:8081/artifactory/generic-local/sql-demo/artifact.zip
                """
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh """
                  ansible-playbook -i /ansible/inventory /ansible/deploy.yml \
                  -e artifact_url=${ARTIFACT_URL} \
                  -e target_env=dev
                """
            }
        }

        stage('Approval for Prod') {
            steps {
                script {
                    input message: "Approve deployment to PROD?", ok: "Deploy"
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                sh """
                  ansible-playbook -i /ansible/inventory /ansible/deploy.yml \
                  -e artifact_url=${ARTIFACT_URL} \
                  -e target_env=prod
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check logs."
        }
    }
}
