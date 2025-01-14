pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    triggers {
        githubPush()  // Automatically triggers the pipeline when code is pushed to GitHub
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm  // Checkout the latest code from the repository
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building project...'
                    sh 'mvn clean install'  // Build the project using Maven
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Java \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Java
                    '''  // Run SonarQube analysis
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // Wait for the quality gate and abort pipeline if the quality gate fails
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                    sh 'mvn test'  // Run unit tests

                    // Publish test results from Surefire reports (correct the path)
                    junit '**/target/surefire-reports/*.xml'  // Publish JUnit test results

                    // Optionally, publish HTML test reports
                    publishHTML(target: [
                        reportName: 'Test Results',
                        reportDir: 'target/surefire-reports',  // Ensure this is the correct directory for the HTML report
                        reportFiles: 'surefire-report.html',  // Ensure this is the correct file for the HTML report
                        keepAll: true
                    ])
                }
            }
        }


        stage('Deploy to Beta') {
            when {
                branch 'beta'  // Trigger this stage only if the branch is 'beta'
            }
            steps {
                script {
                    echo 'Deploying to Beta environment...'
                    sh './deploy-to-beta.sh'  // Deploy to Beta environment
                }
            }
        }

        stage('Deploy to OneBox') {
            when {
                branch 'onebox'  // Trigger this stage only if the branch is 'onebox'
            }
            steps {
                script {
                    echo 'Deploying to OneBox environment...'
                    sh './deploy-to-onebox.sh'  // Deploy to OneBox environment
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'prod'  // Trigger this stage only if the branch is 'prod'
            }
            steps {
                script {
                    echo 'Deploying to Prod environment...'
                    sh './deploy-to-prod.sh'  // Deploy to Production environment
                }
            }
        }

        stage('Post-Deployment Actions') {
            steps {
                script {
                    // Send notification to Slack about deployment status
                    slackSend(channel: env.SLACK_CHANNEL, message: "Deployment successful for ${env.BRANCH_NAME} branch.")
                }
            }
        }

        stage('Archive Reports') {
            steps {
                script {
                    // Archive HTML and PDF reports generated during the test phase
                    archiveArtifacts allowEmptyArchive: true, artifacts: '**/target/*.html, **/target/*.pdf', followSymlinks: false
                }
            }
        }
    }

    post {
        success {
            // Send a success notification to Slack when the pipeline completes successfully
            slackSend(channel: env.SLACK_CHANNEL, message: "Build ${env.BUILD_ID} completed successfully!")
            echo 'Pipeline completed successfully.'
        }
        failure {
            // Send a failure notification to Slack when the pipeline fails
            slackSend(channel: env.SLACK_CHANNEL, message: "Build ${env.BUILD_ID} failed. Please check the logs!")
            echo 'Pipeline failed.'
        }
    }
}
