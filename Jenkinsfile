pipeline {
    agent {
	    node {
	    label 'jenkins-slave'
        }
	    AWS_ACCOUNT_ID="763628714830"
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="fallenmaverick"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
		PATH = "$PATH:/opt/maven/bin"
    }
   
    stages {
        
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${ap-south-1} | docker login --username AWS --password-stdin ${763628714830}.dkr.ecr.${ap-south-1}.amazonaws.com"
                }
                 
            }
        }
        
        stage('GetCode'){
            steps{
                git 'https://github.com/fallenmaverick/java-login.git'
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
                dockerImage = docker.build "${fallenmaverick}:${latest}"
                }
            }
        }
   
    // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{  
                script {
                sh "docker tag ${fallenmaverick}:${latest} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${763628714830}.dkr.ecr.${ap-south-1}.amazonaws.com/${fallenmaverick}:${latest}"
                }
            }
        }
	  
	stage("Deploy to EKS"){
        steps{
	  
		withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', serverUrl: '') {
          		sh '''if /var/lib/jenkins/bin/kubectl get deploy | grep java-login-app
            		then
            		/var/lib/jenkins/bin/kubectl set image deployment fallenmaverick java-app=${763628714830}.dkr.ecr.${ap-south-1}.amazonaws.com/${fallenmaverick}:${latest}
            		/var/lib/jenkins/bin/kubectl rollout restart deployment java-login-app
            		else
		    	    /var/lib/jenkins/bin/kubectl apply -f deployment.yaml
            		fi'''
			    }
			
		    }
        }
    
    stage("Wait for Deployments") {
        steps {
            timeout(time: 2, unit: 'MINUTES') {
            sh '/var/lib/jenkins/bin/kubectl get svc'
            }
        }
    }  
}
