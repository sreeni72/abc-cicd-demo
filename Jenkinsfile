pipeline {
    agent any
     
	environment {
		MULE_CRED = credentials('mule.credentials')
		NEXUS_CRED = credentials('nexus.credentials')
		TARGET_ENV = "${Target}"
		BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
		group_id = readMavenPom().getGroupId()
		artifact_id = readMavenPom().getArtifactId()
		version = readMavenPom().getVersion()
		art_name = readMavenPom().getName()
		report_file = "${WORKSPACE}/index.html"
			   	    
	    BUILD_NUMBER = currentBuild.getNumber()
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "maven-snapshots"
        NEXUS_CREDENTIAL_ID = "nexus.credentials"
        
    }
	
    stages {
		stage('Build Application') {
			script{
				echo "Set Variables for ${brance_name}......"
				if(env.BRANCH_NAME == "develop") {
					environment = "dev"
					workerType = "Micro"
					workers = 1						
				}
				if(env.TARGET_ENV == "test") {
					environment = "test"
					workerType = "Micro"
					workers = 1						
				}
				if(env.TARGET_ENV == "stage") {
					environment = "stage"
					workerType = "Micro"
					workers = 1						
				}
				if(env.TARGET_ENV == "cert") {
					environment = "cert"
					workerType = "Micro"
					workers = 1						
				}
				if(env.TARGET_ENV == "prod") {
					environment = "prod"
					workerType = "Micro"
					workers = 1						
				}						
			}
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
				echo "Deploying to CLOUDHUB Starting....."
				bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=${DworkerType} -Dworkers=${workers} -Denvironment=${environment}'
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
