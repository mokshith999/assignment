pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        SONAR_SCANNER = tool 'sonar-scanner'
    }

    stages {

        stage('Validate Branch') {
            when {
                expression { return env.GIT_BRANCH ==~ /.*name\.developer.*/ }
            }
            steps {
                echo "Branch matches name.developer — proceeding"
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        ${SONAR_SCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=jenkins \
                        -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Application') {
            steps {
                echo "Quality Gate passed — building application"
                sh "mvn clean package"
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}

