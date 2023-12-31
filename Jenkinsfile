pipeline {
    agent any
    
    environment {
        credentials = credentials('jenkins-ec2-deployer');
        AWS_ACCESS_KEY_ID="${credentials_USR}"
        AWS_SECRET_ACCESS_KEY="${credentials_PSW}"
        AWS_DEFAULT_REGION='us-east-1'
        AWS_REPO_ECR='255091851557.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_TAG= 1.1
        IMAGE_NAME="banco-ripley-prueba"
        AWS_CLUSTER_NAME='bripley-eks-cluster'
 
        
    }
   options {
           timestamps()
           timeout(time: 20, unit: 'MINUTES')
           disableConcurrentBuilds()
   }
   stages {
            
        stage('Login to ECR-AWS'){
                steps {
                    script {

                        sh """
                        {
                        aws ecr get-login-password --region $AWS_DEFAULT_REGION | aws ecr describe-repositories --repository-names=${IMAGE_NAME} --query 'repositories[*]|[0].repositoryUri' --region ${AWS_DEFAULT_REGION}
                        } 
                        """
                    }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                // Compilar y etiquetar la imagen Docker
                script {
                    def imageName = "${AWS_REPO_ECR}/${IMAGE_NAME}:${IMAGE_TAG}"
                    def dockerfile = "Dockerfile"
                    
                    // Construir la imagen Docker
                    // docker.build(imageName, "-f ${dockerfile} .")
                    sh """sudo docker build -t ${imageName} -f ${dockerfile} ."""
                
                    // Autenticación en el registro de contenedores de AWS
                    withAWS(credentials: 'jenkins-ec2-deployer') {
                        sh """
                        {
                           sudo aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | sudo docker login --username AWS --password-stdin ${AWS_REPO_ECR}
                           sudo docker push ${AWS_REPO_ECR}/${IMAGE_NAME}:${IMAGE_TAG}
                        }
                        """
                        // Subir la imagen Docker al registro de contenedores de AWS
                        
                    }
                }
            }
        }


        stage('Deploy to Kubernetes') {
                steps {
                    script {
                        sh """

                        echo "######### Apply Deployment / Service #########"
                        sudo aws eks --region ${AWS_DEFAULT_REGION} update-kubeconfig --name ${AWS_CLUSTER_NAME}
                        sudo kubectl apply -f kubernetes/deployment.yaml
                        sudo kubectl apply -f kubernetes/service.yaml

                    """
                }
            }
        }
   }

}