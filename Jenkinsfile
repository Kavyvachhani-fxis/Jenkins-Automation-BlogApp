pipeline {
    agent any
    environment {
        // Define the NVD API key here using the Jenkins credentials store
        NVD_API_KEY = credentials('nvd-api-key') // Ensure you add 'nvd-api-key' in Jenkins credentials
        SONARQUBE_SCANNER_HOME = tool name: 'SonarQubeScanner', type: 'ToolType' // Make sure SonarQube Scanner is configured
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout from the main branch of the repository
                git branch: 'main', url: 'https://github.com/Kavyvachhani-fxis/Jenkins-Automation-BlogApp.git'
            }
        }

        stage('Dependency Check') {
            steps {
                script {
                    // Run Dependency-Check with NVD API key
                    echo "Running Dependency-Check with NVD API key..."
                    sh """
                        docker run --rm -v \$WORKSPACE:/src -v \$WORKSPACE/reports:/report \
                          owasp/dependency-check --project CAST_AutoSec --scan /src/pom.xml \
                          --out /report --format ALL --nvdApiKey \$NVD_API_KEY
                    """
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    // Run Trivy scan to scan for container vulnerabilities
                    echo "Running Trivy Scan for container vulnerabilities..."
                    sh 'docker run --rm -v $WORKSPACE:/src aquasec/trivy --no-progress filesystem /src'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube scan
                    echo "Running SonarQube Analysis..."
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            mvn clean install sonar:sonar
                        '''
                    }
                }
            }
        }

        stage('Publish Reports') {
            steps {
                script {
                    // Publish HTML reports for Dependency-Check
                    echo "Publishing Dependency-Check Report..."
                    publishHTML(target: [
                        reportDir: 'reports',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'Dependency Check Report'
                    ])
                }
            }
        }

        stage('Send Email Notification') {
            steps {
                script {
                    // Send an email with the reports and the status of the pipeline
                    echo "Sending email notification with results..."
                    emailext(
                        to: 'your-email@example.com',  // Change to your desired recipient
                        subject: "CAST AutoSec Pipeline - Security Scan Results",
                        body: """
                            The security scan for the CAST AutoSec project is complete.
                            
                            Please find attached the detailed security scan report.
                            
                            Best regards,
                            Jenkins CI/CD Pipeline
                        """,
                        attachmentsPattern: 'reports/*', // Attach reports from the workspace
                        replyTo: 'no-reply@yourcompany.com' // Modify as needed
                    )
                }
            }
        }

        stage('Post Build Cleanup') {
            steps {
                script {
                    // Clean the workspace after the build to free up space
                    echo "Cleaning up workspace..."
                    cleanWs(cleanWhenFailure: false)
                }
            }
        }
    }
    post {
        always {
            // Additional steps to run always after the pipeline execution
            echo "Pipeline execution finished."
        }
        success {
            // Steps to run in case the pipeline was successful
            echo "Pipeline succeeded!"
        }
        failure {
            // Steps to run in case the pipeline failed
            echo "Pipeline failed. Please review the error logs."
        }
    }
}
