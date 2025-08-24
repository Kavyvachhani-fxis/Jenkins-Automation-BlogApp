pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = 'squ_690080621f71ee9419d737f6c43ffb96f18da7a3'
        REPORT_FILE = 'Security_Report.pdf'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Specify the actual GitHub repository URL for checkout
                git url: 'https://github.com/Kavyvachhani-fxis/Jenkins-Automation-BlogApp.git'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Running build process'
                sh 'mvn -B -DskipTests clean package || true'
            }
        }
        
        stage('Dependency-Check') {
            steps {
                echo 'Running Dependency-Check'
                sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/report owasp/dependency-check --project cast-autosec --scan /src --out /report --format ALL'
            }
        }
        
        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy Scan'
                sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/out aquasec/trivy fs --security-checks vuln,config --severity HIGH,CRITICAL --format json -o /out/trivy.json /src'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis'
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -B -DskipTests sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN || true'
                }
            }
        }
        
        stage('ZAP Scan') {
            steps {
                echo 'Running ZAP Security Scan'
                sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/out ghcr.io/zaproxy/zaproxy:latest -daemon -port 8090 -host 0.0.0.0'
            }
        }

        stage('Generate PDF Report') {
            steps {
                echo 'Generating PDF Report'
                // Add your report generation logic here (e.g., using a Python script)
            }
        }

        stage('Publish Reports') {
            steps {
                echo 'Archiving reports'
                archiveArtifacts artifacts: 'reports/**/*.*', fingerprint: true
            }
        }
    }
}
