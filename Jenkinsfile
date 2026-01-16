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

                stage('Upload to Artifactory') {
                    steps {
                        dir('sample-app') {
                            script {
                                def server = Artifactory.server('artifactory')

                                // Create Build Info object
                                def buildInfo = Artifactory.newBuildInfo()
                                buildInfo.env.capture = true

                                // Upload to your "pipeline" repo
                                def uploadSpec = """{
                                  "files": [
                                    {
                                      "pattern": "target/*.jar",
                                      "target": "pipeline/sample-app/"
                                    }
                                  ]
                                }"""

                                // Attach upload to build info
                                server.upload(spec: uploadSpec, buildInfo: buildInfo)

                                // Publish build info so it appears in JFrog
                                server.publishBuildInfo(buildInfo)
                            }
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
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
