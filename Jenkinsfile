pipeline {
    agent any
     
	environment {
		MULE_CRED = credentials('mule.credentials')
		// NEXUS_CRED = credentials('nexus.credentials')
		TARGET_ENV = "${Target}"
		BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
		//group_id = readMavenPom().getGroupId()
		//artifact_id = readMavenPom().getArtifactId()
		//version = readMavenPom().getVersion()
		//art_name = readMavenPom().getName()
		//report_file = "${WORKSPACE}/index.html"
			   	    
	    BUILD_NUMBER = currentBuild.getNumber()
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "maven-snapshots"
         
        
    }
	
    stages {
		stage('Build Application') {
			
			steps {
				bat 'mvn clean package'
			}
		}
		stage("Perfrom MUnit Test"){
				steps{
					bat 'mvn clean test -X'
				}
			}
		stage('Publish to Exchange'){
			steps{
				script{
					if(env.BRANCH_NAME == "develop") 
						bat 'mvn clean deploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -s settings.xml'												
				}
			}
		}
		stage('Deploy to CLOUDHUB'){
			steps{
				script{
					if(env.BRANCH_NAME == "develop") 
					bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=dev'
					 
					if(env.BRANCH_NAME == "release" && env.TARGET_ENV == "test")
					bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=test'
					
					 
					if(env.BRANCH_NAME == "release" && env.TARGET_ENV == "stage")
					bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=stage'
					
					 
					if(env.BRANCH_NAME == "release" && env.TARGET_ENV == "cert")
					bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=cert'
                    			
					if(env.BRANCH_NAME == "master") 
					bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=prod' 				
				}				
			}
		}
		stage("Perform Regression Test"){
			steps {
				script {
					bat 'npm install -g newman'
					try{
						bat 'npm run app-tests-dev' 
						currentBuild.result = 'SUCCESS'
					}
					catch(Exception e){
						currentBuild.result = 'FAILURE'
					}
					junit 'newman.xml'
				}                
			}
		}
    stage ('Clean Workspace') {
			steps {
				cleanWs deleteDirs: true, notFailBuild: true
			}
		}								
	}
	
	post {
		failure {
			cleanWs deleteDirs: true, notFailBuild: true
		}
	} 
}	
