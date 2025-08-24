pipeline {
    agent any

    environment {
        GITHUB_REPO = "https://github.com/Kavyvachhani-fxis/Jenkins-Automation-BlogApp.git"
        SONAR_URL = "http://sonarqube:9000" // Replace with your SonarQube URL
        EMAIL_RECIPIENT = "d23it180@charusat.edu.in"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${GITHUB_REPO}"
            }
        }

        stage('Build') {
            steps {
                script {
                    // Assuming Maven for building Java projects
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Dependency Check') {
            steps {
                script {
                    sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/report owasp/dependency-check --project "CAST AutoSec" --scan /src --out /report --format ALL'
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/out aquasec/trivy fs --security-checks vuln,config --severity HIGH,CRITICAL --format json -o /out/trivy.json /src'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh 'mvn sonar:sonar -Dsonar.host.url=${SONAR_URL} -Dsonar.projectKey=CAST_AutoSec'
                    }
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
                script {
                    // Example of sending an email with the generated report
                    emailext subject: "CAST AutoSec Pipeline Report", 
                             body: "The security scan has completed. Please find the attached report.", 
                             to: "${EMAIL_RECIPIENT}", 
                             attachLog: true
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean up workspace after job completes
        }
    }
}
