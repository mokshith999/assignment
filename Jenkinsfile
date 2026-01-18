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

        stage('Upload & Sonar in Parallel') {
            parallel {

                /* -------------------------------
                   JFrog Upload (Self-hosted)
                -------------------------------- */
                stage('Upload to JFrog') {
                    steps {
                        withCredentials([
                            usernamePassword(
                                credentialsId: 'jfrog-creds',
                                usernameVariable: 'JFROG_USER',
                                passwordVariable: 'JFROG_PASS'
                            )
                        ]) {
                            sh '''
                                echo "Uploading WAR to JFrog..."

                                WAR_FILE=$(ls sample-app/target/*.war)

                                curl --fail -u "$JFROG_USER:$JFROG_PASS" \
                                     -T "$WAR_FILE" \
                                     "http://16.171.145.173:8081/artifactory/pipeline/${JOB_NAME}-${BUILD_NUMBER}-sample.war"
                            '''
                        }
                    }
                }

                /* -------------------------------
                   SonarQube Analysis
                -------------------------------- */
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
                                    -Dsonar.token=$SONAR_AUTH_TOKEN
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
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

