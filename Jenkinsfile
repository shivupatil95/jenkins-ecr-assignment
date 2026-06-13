pipeline {
    agent { label 'docker-agent' }

    environment {
        GIT_REPO       = 'https://github.com/shivupatil95/jenkins-ecr-assignment.git'
        AWS_REGION     = 'us-east-1'
        AWS_ACCOUNT_ID = '894103237817'
        ECR_REPO_NAME  = 'jenkinsecr-assignment'
        IMAGE_TAG      = "${BUILD_NUMBER}"
        IMAGE_URI      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    echo "Building Java application..."
                    mvn clean -B -Denforcer.skip=true package
                '''
            }
        }

        stage('Verify IAM Role') {
            steps {
                sh '''
                    echo "Verifying IAM Role..."
                    aws sts get-caller-identity
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    echo "Logging into ECR..."
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_URI} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    echo "Pushing Docker image..."
                    docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    echo "Deploying image ${IMAGE_URI} to EKS..."
        
                    kubectl set image deployment/java-app \
                    java-app=${IMAGE_URI} \
                    -n dev
        
                    kubectl rollout status deployment/java-app \
                    -n dev \
                    --timeout=300s
                '''
            }
        }
        
        stage('Cleanup') {
            steps {
                sh '''
                    echo "Cleaning Docker resources..."
                    docker system prune -f
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Image pushed -> ${IMAGE_URI}"
        }
        failure {
            echo "FAILURE: Pipeline failed."
        }
    }
}
