pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS-24.6'
    }
    
    environment {
        APP_NAME = 'gallery-app'
        RENDER_URL = 'https://gallery-pxfl.onrender.com'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timeout(time: 10, unit: 'MINUTES')
        skipStagesAfterUnstable()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm ci --production=false'
            }
        }
        
        stage('Code Quality') {
            parallel {
                stage('Lint') {
                    steps {
                        sh 'npm run lint || echo "No linting configured"'
                    }
                }
                stage('Security Audit') {
                    steps {
                        sh 'npm audit --audit-level=moderate --production || true'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    // Publish test results if you have them
                    publishTestResults testResultsPattern: 'test-results.xml'
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                echo "Deploying ${env.APP_NAME} via Git push to Render..."
                
                // Simple health check after deployment
                sleep time: 45, unit: 'SECONDS'
                
                script {
                    def response = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' ${env.RENDER_URL}",
                        returnStdout: true
                    ).trim()
                    
                    if (response == '200') {
                        echo "✅ Application is healthy"
                    } else {
                        error "❌ Health check failed - HTTP ${response}"
                    }
                }
            }
        }
    }
    
    post {
        success {
            script {
                if (env.BRANCH_NAME == 'master') {
                    slackSend(
                        webhookToken: credentials('slack-webhook'),
                        channel: '#deployments',
                        color: 'good',
                        message: "✅ ${env.APP_NAME} deployed successfully!\n" +
                                "Build: #${env.BUILD_NUMBER}\n" +
                                "URL: ${env.RENDER_URL}"
                    )
                }
            }
        }
        
        failure {
            slackSend(
                webhookToken: credentials('slack-webhook'),
                channel: '#deployments',
                color: 'danger',
                message: "❌ ${env.APP_NAME} build failed!\n" +
                        "Build: #${env.BUILD_NUMBER}\n" +
                        "Logs: ${env.BUILD_URL}"
            )
        }
        
        cleanup {
            cleanWs()
        }
    }
}