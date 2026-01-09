pipeline {
    agent any

    tools {
        jdk 'jdk17'
    }

    triggers {
        githubPush()
    }

    environment {
        SONAR_SCANNER = tool 'sonar-scanner'
        ARTIFACTORY_SERVER = 'artifactory'
        ARTIFACTORY_REPO = 'libs-release-local'
    }

    stages {

        stage('Validate Branch') {
            when {
                expression { return env.BRANCH_NAME ==~ /.*name\.developer.*/ }
            }
            steps {
                script {
                    // Extract developer name from branch
                    env.DEV_NAME = env.BRANCH_NAME.split('\\.')[1]
                    echo "Developer
