pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Nirajkumar18/react-landing-page.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Create Deployment Package') {
            steps {
                sh 'zip -r dist.zip dist appspec.yml scripts'
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(credentials: 'aws-access-key', region: 'us-east-1') {
                    s3Upload(bucket: 'sept4-bucket', file: 'dist.zip')
                }
            }
        }

        stage('Deploy via CodeDeploy') {
            steps {
                withAWS(credentials: 'aws-access-key', region: 'us-east-1') {
                    script {
                        echo "Initiating CodeDeploy deployment..."
                        def deploymentId = awsCodeDeploy(
                            applicationName: 'my-app',
                            deploymentGroupName: 'myapp-deploy-grp',
                            s3Location: [
                                bucket: 'sept4-bucket',
                                key: 'dist.zip',
                                bundleType: 'zip'
                            ]
                        )
                        echo "Deployment initiated with ID: ${deploymentId}"

                        // Check deployment status using AWS CLI
                        sh "aws deploy get-deployment --deployment-id ${deploymentId} --region us-east-1"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully."
        }

        failure {
            echo "Deployment failed. Checking logs..."
            // Optionally, add commands to fetch logs or diagnose failure
        }
    }
}
