pipeline {

    agent any

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: '1.0',
            description: 'Docker/ECR image tag: 1.0, 1.1, 1.2'
        )
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPOSITORY = 'team-java-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
                    export PATH=$JAVA_HOME/bin:$PATH

                    echo "Using Java:"
                    java -version

                    echo "Using Javac:"
                    javac -version

                    chmod +x mvnw
                    ./mvnw clean package -DskipTests
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} .
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

                    ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

                    aws ecr get-login-password \
                        --region ${AWS_REGION} | \
                        docker login \
                        --username AWS \
                        --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Docker Tag') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

                    ECR_IMAGE="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}"

                    docker tag \
                        ${ECR_REPOSITORY}:${IMAGE_TAG} \
                        ${ECR_IMAGE}:${IMAGE_TAG}
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

                    ECR_IMAGE="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}"

                    docker push ${ECR_IMAGE}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo "Image ${IMAGE_TAG} pushed successfully to ECR."
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
