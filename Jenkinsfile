pipeline {
    agent any

    // (Optional) you can also put polling here: triggers { pollSCM('H/1 * * * *') }
    stages {
        stage('Checkout') {
            steps {
                // if this job is "Pipeline from SCM", Jenkins checks out automatically.
                // 'checkout scm' keeps it explicit and works either way.
                checkout scm
            }
        }

        stage('Build Artifact') {
            steps {
                sh '''
                  set -e
                  rm -rf artifact
                  mkdir -p artifact
                  cp hello.sql artifact/
                  # Create a single, stable artifact name
                  cd artifact && zip -r ../sql-demo.zip .
                '''
            }
        }

        stage('Publish Artifact') {
            steps {
                // keep the zip as a Jenkins build artifact
                archiveArtifacts artifacts: 'sql-demo.zip', fingerprint: true

                echo "Artifact URL:"
                echo "${env.BUILD_URL}artifact/sql-demo.zip"
                // For Ansible, you'll usually use 'lastSuccessfulBuild' URL:
                echo "Stable latest URL:"
                echo "${env.JOB_URL}lastSuccessfulBuild/artifact/sql-demo.zip"
            }
        }
    }
}
