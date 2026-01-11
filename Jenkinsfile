pipeline {
    agent any 
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '992382545251'
        IMAGE_REPO_NAME = 'calculator-app'
        IMAGE_TAG = "${env.CHANGE_ID ? 'pr-' + env.CHANGE_ID : 'build-' + env.BUILD_NUMBER}"
        ECR_REGISTRY_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        FULL_IMAGE_NAME = "${ECR_REGISTRY_URL}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Test & Health Check') {
            steps {
                script {
                    echo "Running tests inside the container..."
                    // הרצת הטסטים בתוך קונטיינר זמני
                    sh "docker run --rm ${IMAGE_REPO_NAME}:${IMAGE_TAG} python -m unittest discover -s tests -v"
                }
            }
        }

        // שלב PR - ירוץ רק אם מדובר ב-Pull Request
        stage('PR Verification') {
            when {
                changeRequest() 
            }
            steps {
                echo "This is a PR. Extra verifications can go here."
                // כאן אפשר להוסיף בדיקות לינטר או בדיקות אבטחה
            }
        }

        stage('Login & Push to ECR') {
            steps {
                // שימוש ב-withCredentials לביטחון (כמו שעשית)
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    script {
                        // 1. התחברות ל-ECR
                        sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY_URL}"
                        
                        // 2. תיוג מחדש לכתובת של ECR
                        sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}"
                        
                        // 3. דחיפה
                        sh "docker push ${FULL_IMAGE_NAME}"
                        
                        echo "Image pushed successfully: ${FULL_IMAGE_NAME}"
                    }
                }
            }
        }
    }
    
    // ניקוי בסוף הריצה כדי לחסוך מקום
    post {
        always {
            sh "docker rmi ${IMAGE_REPO_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${FULL_IMAGE_NAME} || true"
        }
    }
}
