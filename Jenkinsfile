pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    // Run the build
                    sh 'mvn clean install'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run tests and show results
                    sh 'mvn test'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                script {
                    // Ensure the artifact exists before archiving it
                    archiveArtifacts 'target/*.war'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Perform SonarQube analysis
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Slack Notification') {
            post {
                always {
                    // Send Slack notification
                    slackSend(channel: '#your-channel', color: 'good', message: "Build ${currentBuild.currentResult}: ${env.JOB_NAME} - ${env.BUILD_URL}", tokenCredentialId: 'slack-webhook')
                }
            }
        }
    }
}
