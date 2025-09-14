pipeline {
    agent any

    environment {
        ARTIFACT_NAME = "artifact.zip"
        ARTIFACT_VERSION = "${env.BUILD_NUMBER}"
        ARTIFACT_URL = "http://artifactory:8081/artifactory/sql-demo/${env.BUILD_NUMBER}/${ARTIFACT_NAME}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/balalingam007/sql-demo.git'
            }
        }

        stage('Build & Package') {
            steps {
                sh 'zip -r ${ARTIFACT_NAME} src/*.js'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    sh 'sonar-scanner -Dsonar.projectKey=sql-demo -Dsonar.sources=src'
                }
            }
        }

        stage('Publish Artifact') {
            steps {
                sh '''
                curl -u admin:password -T ${ARTIFACT_NAME} \
                "http://artifactory:8081/artifactory/sql-demo/${BUILD_NUMBER}/${ARTIFACT_NAME}"
                '''
            }
        }

        stage('Deploy to Dev') {
            when {
                expression { env.BRANCH_NAME.startsWith("feature/") }
            }
            steps {
                sh '''
                ansible-playbook -i /ansible/inventory.ini /ansible/deploy.yml \
                -e "artifact_version=${BUILD_NUMBER}" --limit dev
                '''
            }
        }

        stage('Approval for Prod') {
            when {
                branch 'release'
            }
            steps {
                input message: "Approve deployment to PROD?"
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'release'
            }
            steps {
                sh '''
                ansible-playbook -i /ansible/inventory.ini /ansible/deploy.yml \
                -e "artifact_version=${BUILD_NUMBER}" --limit prod
                '''
            }
        }
    }
}
