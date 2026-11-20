pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
                echo 'Source code checked out successfully.'
            }
        }

        stage('Validate Project Files') {
            steps {
                sh '''
                    echo "Checking website files..."

                    test -f index.html
                    test -f about.html
                    test -f contact.html
                    test -f Dockerfile
                    test -f compose.yaml
                    test -d course

                    echo "All required files are present."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building website Docker image..."
                    docker compose -p devops build web
                '''
            }
        }

        stage('Deploy Websites') {
            steps {
                sh '''
                    echo "Deploying websites..."
                    docker compose -p devops up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Verifying website containers..."

                    test "$(docker inspect --format='{{.State.Running}}' devops-web)" = "true"
                    test "$(docker inspect --format='{{.State.Running}}' course-web)" = "true"

                    docker compose -p devops ps

                    echo "Both websites are running successfully."
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Both websites were deployed successfully!'
        }

        failure {
            echo 'FAILURE: Pipeline failed. Check the Console Output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
