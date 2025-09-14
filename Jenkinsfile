pipeline {
    agent any

    environment {
        SONARQUBE = 'SonarQube'  // Jenkins SonarQube server config name
        ARTIFACTORY = 'Artifactory' // Jenkins Artifactory config name
        REPO = 'sql-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: env.BRANCH_NAME, url: 'https://github.com/balalingam007/sql-demo.git'
            }
        }

        stage('Build & Package') {
            steps {
                sh '''
                mkdir -p build
                zip -r build/artifact.zip *.sql
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/artifact.zip', fingerprint: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=${REPO} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Publish to Artifactory') {
            steps {
                sh '''
                curl -u $ARTIFACTORY_USER:$ARTIFACTORY_PASS \
                    -T build/artifact.zip \
                    "http://artifactory:8081/artifactory/libs-release-local/${REPO}/${BRANCH_NAME}/artifact.zip"
                '''
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh '''
                ansible-playbook -i /ansible/inventory.ini /ansible/deploy.yml \
                    -e "artifact_url=http://artifactory:8081/artifactory/libs-release-local/${REPO}/${BRANCH_NAME}/artifact.zip" \
                    --limit dev
                '''
            }
        }

        stage('Approval for Prod') {
            when {
                branch 'release'
            }
            steps {
                input message: "Deploy to Prod?", ok: "Approve"
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'release'
            }
            steps {
                sh '''
                ansible-playbook -i /ansible/inventory.ini /ansible/deploy.yml \
                    -e "artifact_url=http://artifactory:8081/artifactory/libs-release-local/${REPO}/${BRANCH_NAME}/artifact.zip" \
                    --limit prod
                '''
            }
        }
    }
}
