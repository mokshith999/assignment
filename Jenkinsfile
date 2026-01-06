pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
                sh 'echo "Build completed successfully"'
            }
        }

        stage('Post-Build') {
            steps {
                echo "Pipeline finished for branch: ${env.BRANCH_NAME}"
            }
        }
    }
}
