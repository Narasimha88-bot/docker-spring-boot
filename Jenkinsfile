pipeline {
    agent { label 'slave_node' }

    environment {
        AWS_REGION   = 'us-east-1'
        ECR_REGISTRY = '183004894778.dkr.ecr.us-east-1.amazonaws.com'
        REPOSITORY   = 'narasimha/sample-repo'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Narasimha88-bot/docker-spring-boot'
            }
        }

        stage('Set Variables') {
            steps {
                script {
                    env.IMAGE_TAG = "${env.GIT_COMMIT.take(7)}-${env.BUILD_NUMBER}"
                    env.IMAGE_URI = "${ECR_REGISTRY}/${REPOSITORY}:${IMAGE_TAG}"
                }
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_URI}")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Helm Package') {
            steps {
                sh '''
                helm lint mychart
                helm package mychart
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                  --region us-east-1 \
                  --name my-eks-cluster

                helm upgrade --install myrelease-21 springboot-0.1.0.tgz \
                  --namespace dev \
                  --create-namespace \
                  --set image.repository=${ECR_REGISTRY}/${REPOSITORY} \
                  --set image.tag=${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully'
        }

        failure {
            echo 'Deployment failed'
        }
    }
}
