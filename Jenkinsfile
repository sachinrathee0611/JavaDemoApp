pipeline {
    agent any

    tools {
        maven 'maven3'
    }
    environment {
        MAVEN_OPTS = '--add-opens java.base/java.lang=ALL-UNNAMED'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        SONARQUBE_SERVER = 'sonar-server' // The name of your SonarQube server
        SLACK_CHANNEL = 'slack-webhook' // Slack channel for notifications
        ARTIFACTS_DIR = "target"  // Directory for generated artifacts
    }

    stages {
        // Stage 1: Checkout the code from the GitHub repository
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Stage 2: Build the project using Maven and generate artifacts
        stage('Build') {
            steps {
                script {
                    echo 'Building the project...'
                    sh 'mvn clean install'  // Build the project using Maven
                }
            }
        }

        // Stage 3: Run SonarQube analysis to check code quality
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Java \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Java
                    '''
                }
            }
        }

        // Stage 4: Wait for the Quality Gate to pass or fail
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
                }
            }
        }

        // Archive generated artifacts for downstream job
        stage('Archive Artifacts') {
            steps {
                script {
                    archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: false
                }
            }
        }
    }

    post {
        success {
            slackSend(channel: SLACK_CHANNEL, message: "Upstream Build ${env.BUILD_ID} completed successfully!")
        }
        failure {
            slackSend(channel: SLACK_CHANNEL, message: "Upstream Build ${env.BUILD_ID} failed. Please check the logs!")
        }
    }
}
