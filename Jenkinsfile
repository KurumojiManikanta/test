pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'rdvan'
        PATH = "${env.HOME}/.local/bin:${env.PATH}"  // Add pip user bin to PATH
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Setup AWS CLI') {
            steps {
                sh '''
                    echo "🔧 Checking AWS CLI..."
                    # Install pip3 if missing
                    if ! command -v pip3 >/dev/null 2>&1; then
                        echo "pip3 not found — installing..."
                        sudo apt-get update -y 
                        sudo apt-get install -y python3-pip 
                    fi

                    
                    if ! command -v aws >/dev/null 2>&1; then
                        echo "AWS CLI not found — installing via pip..."
                        python3 -m pip install --user awscli
                    else
                        echo "✅ AWS CLI already installed."
                    fi
                    
                    aws --version
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        echo "🚀 Uploading files to S3 bucket: ${S3_BUCKET}"
                        export AWS_DEFAULT_REGION=${AWS_REGION}

                        # Upload project files (ignore .git & Jenkinsfile)
                        aws s3 sync . s3://${S3_BUCKET}/ \
                            --exclude ".git/*" \
                            --acl public-read

                        echo "✅ Upload completed successfully!"
                    '''
                }
            }
        }
    }
}
