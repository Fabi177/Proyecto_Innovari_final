pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '009661763899.dkr.ecr.us-east-1.amazonaws.com/innovari'
        CLUSTER_NAME = 'innovari-cluster'
        SERVICE_NAME = 'innovari-service'
        TASK_FAMILY = 'innovari-task'
        CONTAINER_NAME = 'innovari-app'
        AWS_CREDENTIALS_ID = 'aws-jenkins-credentials'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Fabi177/Proyecto_Innovari_final.git'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "${AWS_CREDENTIALS_ID}"
                ]]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REPO}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${ECR_REPO}:latest .
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                    docker push ${ECR_REPO}:latest
                """
            }
        }

        stage('Register ECS Task Definition') {
            steps {
                sh """
                    aws ecs register-task-definition \
                        --cli-input-json file://container.json \
                        --region ${AWS_REGION}
                """
            }
        }

        stage('Update ECS Service') {
            steps {
                sh """
                    aws ecs update-service \
                        --cluster ${CLUSTER_NAME} \
                        --service ${SERVICE_NAME} \
                        --force-new-deployment \
                        --region ${AWS_REGION}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Despliegue completado correctamente"
        }
        failure {
            echo "❌ Hubo un error en el pipeline"
        }
    }
}
