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
                expression { return env.BRANCH_NAME ==~ /.*developer.*/ }
            }
            steps {
                script {
                    echo "Branch matches developer pattern — pipeline will run"

                    // Extract developer name safely
                    // Example branch: mokshith.developer → DEV_NAME = mokshith
                    env.DEV_NAME = env.BRANCH_NAME.split('\\.')[0]
                    echo "Developer Name: ${env.DEV_NAME}"
                }
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
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=. \
                        -Dsonar.host.url=$SONAR_HOST_URL
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
                sh "cd sample-app && mvn clean package"
            }
        }

        stage('Upload to Artifactory') {
            steps {
                script {
                    def server = Artifactory.server(ARTIFACTORY_SERVER)

                    // Build upload spec as Groovy map (correct format)
                    def uploadSpec = [
                        files: [
                            [
                                pattern: "sample-app/target/*.war",
                                target: "${ARTIFACTORY_REPO}/${env.DEV_NAME}/${env.BUILD_NUMBER}/"
                            ]
                        ]
                    ]

                    echo "Uploading to: ${ARTIFACTORY_REPO}/${env.DEV_NAME}/${env.BUILD_NUMBER}/"
                    server.upload(uploadSpec)
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'sample-app/target/*.war', fingerprint: true
            }
        }
    }
}
