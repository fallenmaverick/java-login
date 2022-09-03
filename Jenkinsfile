pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="503593019758"
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="jenkins-pipeline-build-demo"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
		PATH = "$PATH:/opt/maven/bin"
    }
   
    stages {
        
         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
        
        stage('GetCode'){
            steps{
                git 'https://github.com/vikramDevPrac/java-login-app.git'
            }
         }
		stage('Build') {
           steps {
        	sh '"mvn" -Dmaven.test.failure.ignore clean install' 
      		}
    	}
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      }
	  
	stage("Deploy to EKS"){
      steps{
	  
		withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', serverUrl: '') {
	  		sh "su - ec2-user"
          		sh "/home/ec2-user/bin/kubectl apply -f deployment.yaml"
		}
      }
    }
    stage("Wait for Deployments") {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          sh '/home/ec2-user/bin/kubectl get svc'
        }
      }
    }  
    }
}
