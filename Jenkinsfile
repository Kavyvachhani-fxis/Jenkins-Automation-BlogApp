pipeline {
    agent any

    environment {
        NVD_API_KEY = '9713571f-5580-43b2-b077-c6898780c4d8' // TEMPORARY hardcoded key for testing only
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kavyvachhani-fxis/Jenkins-Automation-BlogApp.git'
            }
        }

        stage('Run Dependency-Check') {
            steps {
                script {
                    echo "Running Dependency-Check with hardcoded NVD API key..."
                    sh """
                        mkdir -p reports
                        docker run --rm -v \$WORKSPACE:/src -v \$WORKSPACE/reports:/report \
                          owasp/dependency-check \
                          --project CAST_AutoSec \
                          --scan /src \
                          --out /report \
                          --format ALL \
                          --nvdApiKey \$NVD_API_KEY
                    """
                }
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'reports/**/*', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Test pipeline completed."
        }
    }
}
