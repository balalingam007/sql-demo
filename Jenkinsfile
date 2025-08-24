pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/balalingam007/sql-demo.git'
            }
        }
        stage('Build Artifact') {
            steps {
                script {
                    sh 'mkdir -p artifact && cp hello.sql artifact/'
                    sh 'zip -r sql-demo.zip artifact/'
                }
            }
        }
        stage('Publish Artifact') {
            steps {
                archiveArtifacts artifacts: 'sql-demo.zip', fingerprint: true
                echo "Artifact available at: ${env.BUILD_URL}artifact/sql-demo.zip"
            }
        }
    }
}
