pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = "http://artifactory:8081/artifactory/generic-local"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/balalingam007/sql-demo.git'
            }
        }

        stage('Build & Package') {
            steps {
                sh 'zip -r artifact.zip *.sql'
            }
        }

        stage('Publish to Artifactory') {
            steps {
                script {
                    def artifactName = "artifact-${env.BUILD_NUMBER}.zip"
                    sh """
                      curl -u admin:password -T artifact.zip ${ARTIFACTORY_URL}/${artifactName}
                    """
                    env.ARTIFACT_URL = "${ARTIFACTORY_URL}/${artifactName}"
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                script {
                    if (env.BRANCH_NAME.startsWith("feature/")) {
                        echo "Deploying feature branch to DEV..."
                        sh """
                          docker exec ansible-control ansible-playbook \
                          -i /ansible/inventory /ansible/deploy.yml \
                          -e target_env=dev -e artifact_url=${env.ARTIFACT_URL}
                        """
                    } else if (env.BRANCH_NAME.startsWith("release/") || env.BRANCH_NAME == "main") {
                        echo "Deploying release branch to STAGING..."
                        sh """
                          docker exec ansible-control ansible-playbook \
                          -i /ansible/inventory /ansible/deploy.yml \
                          -e target_env=staging -e artifact_url=${env.ARTIFACT_URL}
                        """
                    }
                }
            }
        }

        stage('Approval for Production') {
            when {
                expression { env.BRANCH_NAME == "main" }
            }
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        input message: "Deploy to PRODUCTION?", ok: "Approve"
                    }
                }
            }
        }

        stage('Deploy to Production') {
            when {
                expression { env.BRANCH_NAME == "main" }
            }
            steps {
                echo "Deploying to PRODUCTION..."
                sh """
                  docker exec ansible-control ansible-playbook \
                  -i /ansible/inventory /ansible/deploy.yml \
                  -e target_env=prod -e artifact_url=${env.ARTIFACT_URL}
                """
            }
        }
    }
}
