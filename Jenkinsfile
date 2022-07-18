pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="500279333770"
        AWS_DEFAULT_REGION="us-west-2" 
	CLUSTER_NAME="default"
	SERVICE_NAME="nodejs-container-service"
	TASK_DEFINITION_NAME="first-run-task-definition"
	DESIRED_COUNT="4"
        IMAGE_REPO_NAME="demo"
        IMAGE_TAG="${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
	registryCredential = "demo-admin-user-aws"
	// Override HOME to WORKSPACE
        HOME = "${WORKSPACE}"
        // or override default cache directory (~/.npm)
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
    }
   
    stages {

    // Tests
    stage('Unit Tests Node JS') {
      steps{
        script {
          sh 'npm install'
	  sh 'npm test -- --watchAll=false'
        }
      }
    }
        
    // Building Docker images
    stage('Building Docker image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to AWS ECR') {
     steps{  
         script {
			docker.withRegistry("https://" + REPOSITORY_URI, "ecr:${AWS_DEFAULT_REGION}:" + registryCredential) {
                    	dockerImage.push()
                	}
         }
        }
      }
      
    stage('Deploy AWS ECS') {
     steps{
            withAWS(credentials: registryCredential, region: "${AWS_DEFAULT_REGION}") {
                script {
			sh './script.sh'
                }
            } 
        }
      }      
      
    }
}

