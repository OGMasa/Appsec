pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh '''
                    docker build \
                    --pull \
                    --rm \
                    -f Dockerfile \
                    -t blog:latest .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    rm -f "$WORKSPACE/trivy-report.txt"

                    trivy image \
                    --format table \
                    --output "$WORKSPACE/trivy-report.txt" \
                    blog:latest
                '''
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    odcInstallation: 'OWASP-DC',
                    additionalArguments: '--scan .'
                )
            }
        }

        stage('Run') {
            steps {
                sh '''
                    docker stop blog || true
                    docker rm blog || true

                    docker run -d \
                    -p 3000:3000 \
                    --name blog \
                    blog:latest

                    sleep 5
                '''
            }
        }

        stage('Nikto Scan') {
            steps {
                sh '''
                    docker run --rm \
                    --network host \
                    hackllc/nikto \
                    -h http://127.0.0.1:3000
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
        }
    }
}