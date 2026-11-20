pipeline{
    agent any
    stages{
        stage('Checkout Source Code'){
            steps{
                checkout scm
                echo 'Source Code checked out successfully.'
            }
        }

        stage('Validate Project Files'){
            steps{
                sh '''
                echo "Checking Website files..."

                test -f index.html
                test -f about.html
                test -f contact.html

                echo "Checking Docker configuration..."

                test -f Dockerfile
                test -f compose.yaml

                echo "All required files are present."
                '''
            }
        }

        stage('Build Completed'){
            steps{
                echo 'CI pipeline completed successfully.'
            }
        }
    }

    post{
        success{
            echo 'SUCCESS: your project passed the pipeline'
        }

        failure{
            echo 'FAILURE: your project failed the pipeline'
        }
    }
}