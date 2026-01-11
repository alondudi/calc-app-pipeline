pipeline {
    agent none 

    environment {
        AWS_ACCOUNT_ID = '992382545251'
        AWS_DEFAULT_REGION = 'us-east-1'
        IMAGE_REPO_NAME = 'alon-repo'
        ECR_REGISTRY_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        IMAGE_TAG = "${env.CHANGE_ID ? 'pr-' + env.CHANGE_ID : 'build'}-${env.BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${ECR_REGISTRY_URL}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
    }

    stages {
        
        stage('Unit Tests') {
            agent {
                docker { image 'python:3.9-slim' }
            }
            steps {
                script {
                    echo "Running tests directly using pytest..."
                    sh 'pip install -r requirements.txt'
                    sh 'pip install pytest'
                    sh 'python -m unittest discover -s tests -v'
                }
            }
        }

        stage('Build and Push') {
            agent any
            steps {
                script {
                    echo "Generating Dockerfile..."
                    def dockerfileContent = """
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "api.py"]
""".stripIndent()
                    writeFile file: 'Dockerfile', text: dockerfileContent

                    echo "Tests passed. Building Docker image..."
                    sh "docker build -t ${FULL_IMAGE_NAME} ."

                    echo "Pushing to ECR..."
                    docker.withRegistry("https://${ECR_REGISTRY_URL}", "ecr:${AWS_DEFAULT_REGION}:aws-creds") {
                        sh "docker push ${FULL_IMAGE_NAME}"
                    }
                }
            }
            post {
                always {
                    sh "docker rmi ${FULL_IMAGE_NAME} || true"
                }
            }
        }
    }
}
