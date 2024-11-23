pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO_PREFIX = 'public.ecr.aws/d1k1o6n7/streamingapp' // Replace with your public ECR repository
        IMAGE_TAG = "${env.BUILD_ID}" // Tag images with the Jenkins build ID
        DOCKER_CREDENTIALS = credentials('ravikishans')
        EKS_CLUSTER_NAME = "streamingapp-eks-cluster"
        ARGOCD_SERVER= "https://a14e8200912ca401db17b015634f9a1b-715262143.ap-south-1.elb.amazonaws.com/applications/argocd/streamingapp?resource=&view=network"
        ARGOCD_APP_NAME= "streamingapp"

        HELM_RELEASE_NAME = "streamingapp"
        HELM_CHART_PATH = './k8s/streamingapp' // Path to Helm chart
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Ravikishans/StreamingApp.git'
            }
        }

        // stage("Configure Environment Files for Backend Services") {
        //     steps {
        //         sh """
        //             echo "MONGO_URI=mongodb://root:example@mongo-svc.db.svc.cluster.local:27017/admin
        //             PORT=3001
        //             AWS_REGION=${AWS_REGION}
        //             AWS_S3_BUCKET=rakshi2502" > ./backend/authService/.env

        //             echo "MONGO_URI=mongodb://root:example@mongo-svc.db.svc.cluster.local:27017/admin
        //             PORT=3002
        //             AWS_REGION=${AWS_REGION}
        //             AWS_S3_BUCKET=rakshi2502" > ./backend/streamingService/.env
        //         """
        //     }
        // }

        // stage('Delete Old Images from ECR') {
        //     steps {
        //         script {
        //             withCredentials([[
        //                 $class: 'AmazonWebServicesCredentialsBinding',credentialsId: 'aws_credentials' // Update with your actual AWS credentials ID in Jenkins
        //             ]]) {
        //                 sh """
        //                     # Authenticate Docker to ECR public
        //                     aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/d1k1o6n7
        //                     # List images in the ECR repository and delete images that are not the latest one
        //                     echo "Listing images in ${ECR_REPO_PREFIX}"
        //                     IMAGE_IDS=\$(aws ecr describe-images --repository-name streamingapp --query "imageIds.imageDigest" --output json)
        //                     if [ -n "\$IMAGE_IDS" ]; then
        //                         echo "Deleting old images"
        //                         aws ecr batch-delete-image --repository-name streamingapp --image-ids imageDigest=\$IMAGE_IDS
        //                     else
        //                         echo "No old images to delete"
        //                     fi
        //                 """
        //             }
        //         }
        //     }
        // }

//
        stage('Build and Push Docker Images') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',credentialsId: 'aws_credentials' // Update with your actual AWS credentials ID in Jenkins
                    ]]) {
                        sh """
                        # Authenticate Docker to ECR public
                        aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/d1k1o6n7

                        # Build Docker images
                        docker compose build
                    """
                    }
                }
            }
        }

        stage('push & tag images') {
            steps {
                script{ 
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',credentialsId: 'aws_credentials'
                    ]]) {
                        sh """
                        # Tag images for ECR
                        docker tag ravikishans/streamingapp:frontend ${ECR_REPO_PREFIX}:frontend
                        docker tag ravikishans/streamingapp:backend_auth ${ECR_REPO_PREFIX}:backend_auth
                        docker tag ravikishans/streamingapp:backend_stream ${ECR_REPO_PREFIX}:backend_stream
                        docker tag mongo ${ECR_REPO_PREFIX}:mongo
                        docker tag mongo-express ${ECR_REPO_PREFIX}:mongo-express
                        # Push images to ECR
                        docker push ${ECR_REPO_PREFIX}:frontend
                        docker push ${ECR_REPO_PREFIX}:backend_auth
                        docker push ${ECR_REPO_PREFIX}:backend_stream
                        docker push ${ECR_REPO_PREFIX}:mongo
                        docker push ${ECR_REPO_PREFIX}:mongo-express
                        """
                    }
                }
            }
        }

        stage('Validate Placeholders in Helm Chart') {
            steps {
                script {
                    sh """
                        grep 'ravikishans/streamingapp:frontend' ${HELM_CHART_PATH}/values.yaml || echo 'Placeholder not found!'
                        grep 'ravikishans/streamingapp:backend_auth' ${HELM_CHART_PATH}/values.yaml || echo 'Placeholder not found!'
                        grep 'ravikishans/streamingapp:backend_stream' ${HELM_CHART_PATH}/values.yaml || echo 'Placeholder not found!'
                    """
                }
            }
        }

        stage('Update Helm Chart with ECR Image Tags') {
            steps {
                script {
                    sh """
                        sed -i "s|ravikishans/streamingapp:frontend|${ECR_REPO_PREFIX}:frontend|g" ${HELM_CHART_PATH}/values.yaml
                        sed -i "s|ravikishans/streamingapp:backend_auth|${ECR_REPO_PREFIX}:backend_auth|g" ${HELM_CHART_PATH}/values.yaml
                        sed -i "s|ravikishans/streamingapp:backend_stream|${ECR_REPO_PREFIX}:backend_stream|g" ${HELM_CHART_PATH}/values.yaml
                        sed -i "s|mongo:latest|${ECR_REPO_PREFIX}:mongo|g" ${HELM_CHART_PATH}/values.yaml
                    """
                }
            }
        }
        stage('value.yaml') {
            steps {
                script {
                    sh """
                       cat ${HELM_CHART_PATH}/values.yaml
                    """   
                }
            }
        }

        stage('eks update') {
            steps {
                script{ 
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',credentialsId: 'aws_credentials'
                    ]]) {
                        sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                        kubectl get pods --all-namespaces
                        kubectl get svc --all-namespaces
                        """
                    }
                }    
            }
        }    

        stage('helm deploy') {
            steps {
                script{ 
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',credentialsId: 'aws_credentials'
                    ]]) {
                        sh """
                        helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} --namespace default --create-namespace
                        """
                    }
                }    
            }
        }

        stage(argocd) {
            steps {
                script{ 
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',credentialsId: 'aws_credentials'
                    ]]) {
                        sh """
                        argocd login ${ARGOCD_SERVER} --username admin --password 123@Argocd --insecure
                        argocd app sync ${ARGOCD_APP_NAME}
                        """
                    }
                }    
            }
        }

        stage('verify deployment') {
            steps {
                script{ 
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',credentialsId: 'aws_credentials'
                    ]]) {
                        sh """
                        kubectl get pods --all-namespaces
                        kubectl get svc --all-namespaces
                        """
                    }
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



// argocd login ${ARGOCD_SERVER} --username admin --password 123@Argocd --insecure 
// argocd app sync ${ARGOCD_APP_NAME}