pipeline {
    agent {
        docker {
            image 'ubuntu:22.04'
            args '-u root'
        }
    }
    stages {
        stage('Install Tools') {
            steps {
                sh 'apt-get update && apt-get install -y zip unzip'
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/balalingam007/sql-demo.git'
            }
        }
        stage('Build Artifact') {
            steps {
                sh 'mkdir -p artifact && cp hello.sql artifact/'
                sh 'zip -r sql-demo.zip artifact/'
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
