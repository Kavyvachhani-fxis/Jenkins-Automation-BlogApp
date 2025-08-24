pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = 'sqv_690080621f71ee94197f6c43ffb96f18da7a3'
        REPORT_FILE = 'Security_Report.pdf'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kavyvachhani-fxis/Jenkins-Automation-BlogApp.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package || true'
            }
        }

        stage('Dependency-Check') {
            steps {
                sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/report owasp/dependency-check --project "CAST AutoSec" --scan /src --out /report --format ALL'
            }
        }

        stage('Trivy') {
            steps {
                sh 'docker run --rm -v $WORKSPACE:/src -v $WORKSPACE/reports:/out aquasec/trivy fs --security-checks vuln,config --severity HIGH,CRITICAL --format json -o /out/trivy.json /src'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn -B -DskipTests sonar:sonar -Dsonar.host.url=http://localhost:9000 || true'
                }
            }
        }

        stage('Publish Reports') {
            steps {
                archiveArtifacts artifacts: 'reports/**/*.*', fingerprint: true
            }
        }
    }
    post {
        always {
            emailext subject: "Build Complete: ${currentBuild.currentResult}",
                     body: "Build status: ${currentBuild.currentResult}\nCheck the logs for more information.",
                     to: 'd23it180@charusat.edu.in',
                     from: 'kavytelecom@gmail.com'
        }
    }
}
