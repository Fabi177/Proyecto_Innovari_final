pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '009661763899.dkr.ecr.us-east-1.amazonaws.com/innovari'
        ECS_CLUSTER = 'innovari-cluster'
        ECS_SERVICE = 'innovari-service'
        TASK_DEFINITION_FAMILY = 'innovari-task'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REPO}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ECR_REPO}:latest", "-f docker/php/Dockerfile .")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push ${ECR_REPO}:latest"
            }
        }

        stage('Register ECS Task Definition') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh """
                        aws ecs register-task-definition \
                            --cli-input-json file://container.json \
                            --region ${AWS_REGION}
                    """
                }
            }
        }

        stage('Update ECS Service') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
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
            echo '✅ Deploy exitoso!'
        }
        failure {
            echo '❌ Hubo un error en el pipeline'
        }
    }
}
