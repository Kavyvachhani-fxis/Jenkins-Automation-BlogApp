pipeline {
    agent any
    environment {
        GITHUB_REPO = 'https://github.com/Kavyvachhani-fxis/Jenkins-Automation-BlogApp.git'
        SONARQUBE_URL = 'http://localhost:9000'
        EMAIL_RECIPIENT = 'd23it180@charusat.edu.in'
        NVD_API_URL = 'https://services.nvd.nist.gov/rest/json/cpes/2.0'
        CPE_MATCH_STRING = 'cpe:2.3:o:microsoft:windows_10'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package || true'
            }
        }

        stage('CPE API Integration') {
            steps {
                script {
                    // Call the NVD CPE API to get CPE records
                    def cpeUrl = "${NVD_API_URL}?cpeMatchString=${CPE_MATCH_STRING}"
                    def response = sh(script: "curl -s ${cpeUrl}", returnStdout: true).trim()
                    
                    // Print out the response for debugging purposes
                    echo "CPE API Response: ${response}"

                    // Parse the response if you need to extract specific details (example: JSON parsing)
                    def cpeJson = readJSON text: response
                    // You can access specific details from the response here
                    echo "Number of CPE records found: ${cpeJson.totalResults}"
                }
            }
        }

        stage('Dependency-Check') {
            steps {
                sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/report owasp/dependency-check --project CAST_AutoSec --scan /src --out /report --format ALL'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/out aquasec/trivy fs --security-checks vuln,config --severity HIGH,CRITICAL --format json -o /out/trivy.json /src'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn -B -DskipTests sonar:sonar -Dsonar.host.url=${SONARQUBE_URL} || true'
                }
            }
        }

        stage('Publish Reports') {
            steps {
                archiveArtifacts artifacts: 'reports/**/*.*', fingerprint: true
            }
        }

        stage('Send Email') {
            steps {
                emailext (
                    subject: 'Security and Build Report: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}',
                    body: '''Build Result: ${BUILD_STATUS}
                    Please find the security scan results attached.
                    Check the console output for details.
                    ''',
                    to: "${EMAIL_RECIPIENT}",
                    attachmentsPattern: 'reports/**/*.pdf',
                    mimeType: 'text/plain'
                )
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after every build
        }
    }
}
