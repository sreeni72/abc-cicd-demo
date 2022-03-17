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
				echo 'Build the Mule Application...' 
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
					if(env.BRANCH_NAME == "develop") {
						echo "Publishing to Exchange Starting....."
						try{
							bat 'mvn clean deploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -s settings.xml'
							currentBuild.result = 'SUCCESS'
							echo "Publishing to Exchange Completed....."
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
							echo "Publishing to Exchange Failed, Artifact already Exits....."
						}						
					}
				}
			}
		}
		stage('Deploy to CLOUDHUB'){
			steps{
				script{
					
					if(env.BRANCH_NAME == "develop") {
						t_environment = "dev"
						t_workerType = "Micro"
						t_workers = 1						
					}
					if(env.TARGET_ENV == "test") {
						t_environment = "test"
						t_workerType = "Micro"
						t_workers = 1						
					}
					if(env.TARGET_ENV == "stage") {
						t_environment = "stage"
						t_workerType = "Micro"
						t_workers = 1						
					}
					if(env.TARGET_ENV == "cert") {
						t_environment = "cert"
						t_workerType = "Micro"
						t_workers = 1						
					}
					if(env.TARGET_ENV == "prod") {
						t_environment = "prod"
						t_workerType = "Micro"
						t_workers = 1						
					}						
				}
				echo "Deploying to CLOUDHUB Starting....."
				bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType="${t_workerType}" -Dworkers="${t_workers}" -Denvironment="${t_environment}"'
				echo "Deploying to CLOUDHUB Completed....." 
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
    }
}	
