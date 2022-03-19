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
        
		//WORKER_TYPE=""
		//WORKERS=""
        
    }
	
    stages {
		stage('Build Application') {			
			steps {
				bat 'mvn clean package'
			}
		}
		stage('Perfrom MUnit Test'){
			when { expression { env.BRANCH_NAME == "develop"} }
			steps{
					bat 'mvn clean test -X'
			}
		}
		stage ('Sonar Analysis') {
			when { expression { env.BRANCH_NAME != "master"} }
            steps {
                echo 'Excecuted other than master branch.'
            }
		}	
		stage('Publish to Exchange'){
			when { expression { env.BRANCH_NAME == "develop"} }
			steps{
				echo "publish"
				//bat 'mvn clean deploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -s mvn-settings.xml'					
			}
		}
		stage('Deploy to CLOUDHUB'){
			steps{
				script{
					if(env.BRANCH_NAME == "develop") {
						TARGET_ENV = "dev"
						WORKER_TYPE = "Micro"
						NO_OF_WORKERS = 1
					}					 
					if(env.BRANCH_NAME == "release" && env.TARGET_ENV == "test"){
						WORKER_TYPE="Micro"
						NO_OF_WORKERS="1"						
					}					 
					if(env.BRANCH_NAME == "release" && env.TARGET_ENV == "stage"){
						WORKER_TYPE="Micro"
						NO_OF_WORKERS="1"
					}					 
					if(env.BRANCH_NAME == "release" && env.TARGET_ENV == "cert"){
						WORKER_TYPE="Micro"
						NO_OF_WORKERS="1"
					}                   			
					if(env.BRANCH_NAME == "master") {
						WORKER_TYPE="Micro"
						NO_OF_WORKERS="1"
					}	
					
					echo "env.TARGET_ENV" + TARGET_ENV
					bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=WORKER_TYPE -Dworkers=1 -Denvironment=TARGET_ENV'	 				
				}		
			}
		}
												
	}
	
	post {
		success {
			cleanWs deleteDirs: true, notFailBuild: true
		}
		failure {
			cleanWs deleteDirs: true, notFailBuild: true
		}
	} 
}	
