pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        AWS_DEFAULT_REGION = 'us-east-1' // Replace with your region
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Clone code from the GitHub repository
                git branch: 'main', url: 'https://github.com/Nirajkumar18/react-landing-page.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install npm dependencies
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                // Build the React application
                sh 'npm run build'
            }
        }

        stage('Create Deployment Package') {
            steps {
                // Create a zip package including build files and deployment scripts
                sh 'zip -r dist.zip dist appspec.yml scripts'
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(credentials: 'aws-access-key') {
                    // Upload the deployment package to S3
                    s3Upload(bucket: 'sept4-bucket', file: 'dist.zip')
                }
            }
        }

        stage('Deploy via CodeDeploy') {
            steps {
                withAWS(credentials: 'aws-access-key') {
                    script {
                        // Start CodeDeploy via AWS CLI
                        sh '''
                            aws deploy create-deployment \
                                --application-name my-app \
                                --deployment-group-name myapp-deploy-grp \
                                --s3-location bucket=sept4-bucket,key=dist.zip,bundleType=zip \
                                --file-exists-behavior OVERWRITE
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Always archive the deployment package as an artifact
            archiveArtifacts artifacts: 'dist.zip', allowEmptyArchive: true

            // Optionally clean up workspace to avoid filling up Jenkins with files
            cleanWs()
        }
    }
}
