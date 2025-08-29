pipeline {
    agent any

    environment {
        // 🔐 Use hardcoded key for now (replace with credentials() later)
        NVD_API_KEY = '9713571f-5580-43b2-b077-c6898780c4d8'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kavyvachhani-fxis/Jenkins-Automation-BlogApp.git'
            }
        }

        stage('Generate config.properties') {
            steps {
                script {
                    echo "Creating config.properties with NVD API key..."
                    writeFile file: 'config.properties', text: "nvd.api.key=${env.NVD_API_KEY}"
                }
            }
        }

        stage('Run Dependency-Check with API Key') {
            steps {
                script {
                    echo "Running OWASP Dependency-Check using official image and mounted config..."
                    sh """
                        mkdir -p reports
                        docker run --rm \
                          -v \$WORKSPACE:/src \
                          -v \$WORKSPACE/reports:/report \
                          -v \$WORKSPACE/config.properties:/usr/share/dependency-check/data/config.properties \
                          ghcr.io/jeremylong/owasp-dependency-check:latest \
                          --project "CAST AutoSec" \
                          --scan /src \
                          --out /report \
                          --format ALL
                    """
                }
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'reports/**/*', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
        failure {
            echo "Pipeline failed. Check logs and reports."
        }
    }
}
