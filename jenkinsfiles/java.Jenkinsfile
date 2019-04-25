@Library('my_shared_library')_

def workspace;
def branch;
def dockerImage;
def props='';
def microserviceName;
def port;
def gitUrl;
def repoName;
def credentials = 'docker-credentials';

node {
    stage('Checkout Code')
    {
	checkout scm
	workspace = pwd() 
	     sh "ls -lat"
	props = readProperties  file: """deploy.properties"""   
    }
    
    stage ('Static Code Analysis')
    { 
	    sonarexec "${props['deploy.sonarqubeserver']}"
    }
    
     stage ('Build and Unit Test Execution')
    {
          testexec "junit testing.."
    }
    
     stage ('Code Coverage')
    { 
        codecoveragexec "${props['deploy.sonarqubeserver']}"
    }
    stage ('create war')
    {
    	mavenbuildexec "mvn build"
    }
    
     stage ('Create Docker Image')
    { 
	     echo 'creating an image'
             dockerImage = dockerexec "${props['deploy.microservice']}"
    }
    
     stage ('Push Image to Docker Registry')
    { 
	     docker.withRegistry('https://registry.hub.docker.com',credentials) {
             dockerImage.push("${BUILD_NUMBER}")
             dockerImage.push("latest")
	     }
    }
    
    stage ('Config helm')
    { 
    	sh "echo 'Almost there'"
	sh "echo '${dockerImage}'"
	sh"""
	sed -i 's/nginx/${props['deploy.microservice']}/g' helmchart/values.yaml
	sed -i 's/stable/${BUILD_NUMBER}/g' helmchart/values.yaml
	sed -i 's/80/${props['deploy.port']}/g' helmchart/templates/deployment.yaml
	"""
    }
    /*stage ('deploy to cluster')
    {
    	helmdeploy "${props['deploy.microservice']}"
    }*/
	
}
		
