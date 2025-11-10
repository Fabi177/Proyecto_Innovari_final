pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '009661763899.dkr.ecr.us-east-1.amazonaws.com/innovari'
        IMAGE_TAG = 'latest'
        AWS_CREDENTIALS_ID = 'aws-jenkins-credentials' // ID de las credenciales configuradas en Jenkins
        GIT_REPO = 'https://github.com/Fabi177/Proyecto_Innovari_.git'
        GIT_BRANCH = 'main' // cambiar si tu rama principal tiene otro nombre
        ECS_CLUSTER = 'innovari-cluster'
        ECS_SERVICE = 'innovari-service'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Update ECS Service') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh """
                        aws ecs update-service \
                            --cluster ${ECS_CLUSTER} \
                            --service ${ECS_SERVICE} \
                            --force-new-deployment \
                            --region ${AWS_REGION}
                    """
                }
            }
        }

    }

    post {
        success {
            echo "Pipeline completado con éxito 🚀"
        }
        failure {
            echo "Hubo un error en el pipeline ❌"
        }
    }
}
