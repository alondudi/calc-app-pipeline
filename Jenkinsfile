pipeline {
    agent none 

    environment {
        AWS_ACCOUNT_ID = '992382545251'
        AWS_DEFAULT_REGION = 'us-east-1'
        IMAGE_REPO_NAME = 'alon-repo'
        ECR_REGISTRY_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        IMAGE_TAG = "${env.CHANGE_ID ? 'pr-' + env.CHANGE_ID : (env.BRANCH_NAME == 'main' ? 'prod' : 'build')}-${env.BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${ECR_REGISTRY_URL}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        PROD_EC2_IP = '98.94.21.244' 
        PROD_SSH_ID = 'ssh-prod' // ה-ID של ה-SSH Key שהגדרת בג'נקינס
        APP_PORT = '5000' 
    }

    stages {
        
        stage('Unit Tests') {
            when { anyOf { branch 'main'; changeRequest() } }
            agent { docker { image 'python:3.9-slim' } } 
            steps {
                script {
                    echo "Running tests..."
                    sh 'pip install -r requirements.txt && pip install pytest'
                    sh 'python -m unittest discover -s tests -v'
                }
            }
        }

        stage('Build and Push') {
            when { anyOf { branch 'main'; changeRequest() } }
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
                    
                    sh "docker build -t ${FULL_IMAGE_NAME} ."

                    docker.withRegistry("https://${ECR_REGISTRY_URL}", "ecr:${AWS_DEFAULT_REGION}:aws-creds") {
                        sh "docker push ${FULL_IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Deploy to Prod EC2') {
            when { branch 'main' }
            
            agent { docker { image 'alpine:latest' } } 
            
            steps {
                script {
                    echo "Preparing to deploy to ${PROD_EC2_IP}..."
                    
                    sh 'apk add --no-cache openssh-client'

                    sshagent(credentials: [ssh-prod]) {
                        
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@${PROD_EC2_IP} "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY_URL}"
                        """
                        
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@${PROD_EC2_IP} 'docker pull ${FULL_IMAGE_NAME}'"
                        
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@${PROD_EC2_IP} '
                                docker stop my-app || true
                                docker rm my-app || true
                                docker run -d -p 80:${APP_PORT} --name my-app ${FULL_IMAGE_NAME}
                            '
                        """
                    }
                }
            }
        }

        stage('Health Verification') {
            when { branch 'main' }
            agent { docker { image 'curlimages/curl' } } 
            
            steps {
                script {
                    echo "Verifying service health on http://${PROD_EC2_IP}..."
                    
                    sh """
                        for i in 1 2 3 4 5; do
                            if curl -f http://${PROD_EC2_IP}/health; then
                                echo "Health check passed!"
                                exit 0
                            fi
                            echo "Attempt \$i failed. Retrying in 10s..."
                            sleep 10
                        done
                        echo "Health check failed after 5 attempts."
                        exit 1
                    """
                }
            }
        }
    }
    
    post {
        always {
             node {
                 sh "docker rmi ${FULL_IMAGE_NAME} || true"
             }
        }
    }
}
