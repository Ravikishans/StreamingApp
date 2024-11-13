pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-northeast-2'
        ECR_REPO_PREFIX = 'public.ecr.aws/ravicapstm' // Replace with your public ECR repository
        IMAGE_TAG = "${env.BUILD_ID}" // Tag images with the Jenkins build ID

        HELM_RELEASE_NAME = "streamingapp"
        HELM_CHART_PATH = './k8s/streamingapp' // Path to Helm chart
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Ravikishans/StreamingApp.git'
            }
        }

        stage("Configure Environment Files for Backend Services") {
            steps {
                sh """
                    echo "MONGO_URI=mongodb://root:example@mongo-svc.db.svc.cluster.local:27017/admin
                    PORT=3001
                    AWS_REGION=${AWS_REGION}
                    AWS_S3_BUCKET=rakshi2502" > ./backend/authService/.env

                    echo "MONGO_URI=mongodb://root:example@mongo-svc.db.svc.cluster.local:27017/admin
                    PORT=3002
                    AWS_REGION=${AWS_REGION}
                    AWS_S3_BUCKET=rakshi2502" > ./backend/streamingService/.env
                """
            }
        }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    sh """
                        # Authenticate Docker to ECR public
                        aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws


                        # Build Docker images
                        docker compose build

                        # Tag images for ECR
                        docker tag ravikishans/streamingapp:frontend ${ECR_REPO_PREFIX}/frontend:${IMAGE_TAG}
                        docker tag ravikishans/streamingapp:backend_auth ${ECR_REPO_PREFIX}/backend_auth:${IMAGE_TAG}
                        docker tag ravikishans/streamingapp:backend_stream ${ECR_REPO_PREFIX}/backend_stream:${IMAGE_TAG}

                        # Push images to ECR
                        docker push ${ECR_REPO_PREFIX}/frontend:${IMAGE_TAG}
                        docker push ${ECR_REPO_PREFIX}/backend_auth:${IMAGE_TAG}
                        docker push ${ECR_REPO_PREFIX}/backend_stream:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Helm Chart with ECR Image Tags') {
            steps {
                script {
                    // Replace image tags in Helm values file
                    sh """
                        sed -i 's|frontend-image-tag|${IMAGE_TAG}|g' ${HELM_CHART_PATH}/values.yaml
                        sed -i 's|backend-auth-image-tag|${IMAGE_TAG}|g' ${HELM_CHART_PATH}/values.yaml
                        sed -i 's|backend-stream-image-tag|${IMAGE_TAG}|g' ${HELM_CHART_PATH}/values.yaml
                    """
                }
            }
        }

        stage('Deploy to EKS Using Helm') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    sh """
                        aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME}
                        helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} \
                        --namespace default --create-namespace --kubeconfig=$KUBECONFIG --debug
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    sh "kubectl get pods -n db --kubeconfig=$KUBECONFIG"
                    sh "kubectl get pods -n beauth --kubeconfig=$KUBECONFIG"
                    sh "kubectl get pods -n bestream --kubeconfig=$KUBECONFIG"
                    sh "kubectl get pods -n frontend --kubeconfig=$KUBECONFIG"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
