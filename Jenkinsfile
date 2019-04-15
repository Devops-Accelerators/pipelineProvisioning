properties([parameters([[$class: 'GlobalVariableStringParameterDefinition', defaultValue: '', description: '', name: 'port'], [$class: 'GlobalVariableStringParameterDefinition', defaultValue: '', description: '', name: 'MicroserviceName'], [$class: 'GlobalVariableStringParameterDefinition', defaultValue: '', description: '', name: 'apiRepoURL'], password(defaultValue: '', description: '', name: 'gitPassword'), credentials(credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', defaultValue: '', description: '', name: 'gitCred', required: false)])])
def branchName;
def newJobname;
def addedProjects;
def gitURL;
def props;
def jobsCreated="";
def status;
def ucdjobList;
def microserviceName;
def newjob=true;
def createJIRA=false;
def commit_username;
def commit_Email, repoName, envConfigProp;
node { 
	stage ('Checkout Code')
		{
			checkout scm
        		workspace = pwd ()
			props = readProperties  file: """seedJob.properties"""
       			microserviceName = sh(returnStdout: true, script: """echo ${MicroserviceName} | sed 's/[\\._-]//g'""").trim()
			microserviceName = microserviceName.toLowerCase()
			sh"""echo ${microserviceName}""" 
			commit_username=sh(returnStdout: true, script: '''username=$(git log -1 --pretty=%ae) 
                                                            echo ${username%@*} ''').trim();
			commit_username=sh(returnStdout: true, script: """echo ${commit_username} | sed 's/48236651+//g'""").trim()
			repoName=sh(returnStdout: true, script: """echo \$(basename ${apiRepoURL.trim()})""").trim();
			repoName=sh(returnStdout: true, script: """echo ${repoName} | sed 's/.git//g'""").trim()
			sh"""echo ${repoName}"""
			}
    
 stage ('Create CI Pipeline')
		{				
			createpipelinejob(microserviceName.trim(), apiRepoURL.trim())		
		}
	stage ('Create CD Pipeline')
		{
				
		}
	
	stage ('Add Repo Webhook')
		{
			withCredentials([string(credentialsId: 'githubtoken', variable: 'githubCredentials'),
			usernameColonPassword(credentialsId: 'jenkinsadminCredentials', variable: 'jenkinsAdminCredentials')]) 
			{
				try 
				{
					createGithubWebhook(repoName.trim(), props['jenkins.server'], """${githubAPI}""", """${githubOrg}""", githubCredentials )
				}
				catch (e) 
				{
					currentBuild.result='FAILURE'
					notifyBuild(currentBuild.result, "At Stage Add Repo Webhook", commit_Email, "")
					deletebuildpipeline(microserviceName.trim(), """${jenkinsAdminCredentials}""", props['jenkins.server'].trim(), """${ucdCredentials}""", """${ucdServer}""".trim())
					echo """${e.getMessage()}"""
					throw e
				}
			}
		}
	

	stage ('Add pipeline Scripts to Repository')
		{
			withCredentials([usernameColonPassword(credentialsId: 'jenkinsadminCredentials', variable: 'jenkinsAdminCredentials')]) 
					{
					sh """ rm -rf ${repoName.trim()}
					git clone ${apiRepoURL}""".trim()
					echo "Cloning is done here"
					//add app name and definition file name			
					sh """
					rm -f ${repoName.trim()}/Jenkinsfile
					echo "#second step is done"
					cd ${repoName.trim()}
					 """
					sh """ cd ${repoName.trim()}																
					cp -f ../jenkinsfiles/java.Jenkinsfile Jenkinsfile	
					#change pipeline name in Jenkinsfile
					sed -i 's/pipelineName/${microserviceName.trim()}/g'  Jenkinsfile
					git config --global user.name ${commit_username}
					git init
					git add .
					git commit -m "pipeline Scripts added by seed job"
					git remote rm origin
					git remote add origin ${apiRepoURL}
					git remote -v
 					
					git push -f origin master https://${commit_username}:${gitPassword}@${repoName}
					cd ..
					rm -rf ${repoName.trim()}"""	
			}
		}
}
def createpipelinejob(String jobName, String gitURL)
{
    jobDsl failOnMissingPlugin: true, 
	    sandbox: true,
           scriptText: """pipelineJob("${jobName}") { 
            definition {
                        cpsScm {
                        	scm {
                        		git {
                        			remote {
                        				name('remoteB')
                       					url('${gitURL}')
							credentials('${gitCred}')
                        				}
                        			branch("*/master")
                        			extensions {}	
                        			}	
                        		}
                        		scriptPath("Jenkinsfile")	
                       		}
                        }
                       }"""
}
