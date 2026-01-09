pipeline {
    agent any

    tools {
        jdk 'jdk17'
    }

    triggers {
        githubPush()   // Triggered by GitHub webhook
    }

    environment {
        SONAR_SCANNER = tool 'sonar-scanner'
        ARTIFACTORY_SERVER = 'artifactory'   // Name configured in Jenkins > Manage Credentials
        ARTIFACTORY_REPO = 'libs-release-local'
    }

    stages {

        stage('Validate Branch') {
            when {
                expression { return env.BRANCH_NAME ==~ /.*name\.developer.*/ }
            }
            steps {
                echo "Branch matches name.developer — pipeline will run"
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
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "sample-app/target/*.war",
                                "target": "${ARTIFACTORY_REPO}/sample-app/"
                            }
                        ]
                    }"""
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


