pipeline {
    agent any
    environment {
        WORKSPACE = '/var/lib/jenkins/workspace'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Run Security Scans') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh '''
                        docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/report owasp/dependency-check --project CAST_AutoSec --scan /src/pom.xml --out /report --format ALL --disableNVD
                        '''
                    }
                }
                stage('Trivy Scan') {
                    steps {
                        // Example: run Trivy Scan in parallel
                        sh 'trivy --input /src --output /report/trivy_report.json'
                    }
                }
            }
        }
        stage('Publish Reports') {
            steps {
                // Collect the generated reports and send an email or store in Jenkins
                emailext (
                    subject: "Security Scan Results",
                    body: "The security scans have completed. Check the attached reports.",
                    attachmentsPattern: "reports/*",
                    to: "youremail@example.com"
                )
            }
        }
    }
}
