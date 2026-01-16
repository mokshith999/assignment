pipeline {
    agent any

    tools {
        jdk 'jdk17'
    }

    environment {
        SONARQUBE_ENV = 'sonarqube'
        SONAR_SCANNER = tool 'sonar-scanner'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                dir('sample-app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('sample-app') {
                    withSonarQubeEnv('sonarqube') {
                        sh """
                            ${SONAR_SCANNER}/bin/sonar-scanner \
                            -Dsonar.projectKey=jenkins \
                            -Dsonar.sources=src/main/java \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_AUTH_TOKEN
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
        }
    }
}
