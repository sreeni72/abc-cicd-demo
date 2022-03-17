pipeline {
    agent any
    
	parameters{
	    string CURRENT_BUILD_RESULT = "SUCCESS"
		string(name: "RELEASE_VERSION",description: "current deploy version", defalutValue: "0.0.1-SNAPSHOT")
		booleanParam(name: "CURRENT_BUILD_DEPLOY", defaultValue: false, description: "Deploy application to RTF")
	}
	
    environment {
	
		mule_cred = credentials(' ')
		nexus_cred = credentials(' ')
		brance_name = "${GIT_BRANCH.split("/")[1]}"
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
			steps {
				echo 'Build the Mule Application...' 
				bat 'mvn clean package'
			}
		}
		
		stage('Publish to Exchange'){
			when{ expression { params.CURRENT_BUILD_DEPLOY } }
			steps{
				echo "Publishing to Exchange Starting....."
				withCredentials([usernamePassword(credentialsId: 'mule.credentials', passwordVariable: 'anypoint_pwd', usernameVariable: 'anypoint_user')]) {
    				bat 'mvn clean deploy -Dusername=${anypoint_user} -Dpassword=${anypoint_pwd} -s settings.xml'
    	    			}
				echo "Publishing to Exchange Completed....."
			}
			post{
				success{ echo "Publish to Exchange - Successful" }
				failure{ echo "Publish to Exchange - Failure" }
			}
		}
		
		stage('Deploy to CLOUDHUB'){
			when{ expression { params.CURRENT_BUILD_DEPLOY } }
			steps{
				echo "Deploying to CLOUDHUB Starting....."
				withCredentials([usernamePassword(credentialsId: 'mule.credentials', passwordVariable: 'anypoint_pwd', usernameVariable: 'anypoint_user')]) {
    				bat 'mvn clean deploy -DmuleDeploy -Dusername=${anypoint_user} -Dpassword=${anypoint_pwd} -DworkerType=Micro -Dworkers=1 -Denvironment=dev -Dmule.version=4.4.0'
    	    	}
				echo "Deploying to CLOUDHUB Completed....."
			}
			post{
				success{ echo "Deploy to CLOUDHUB - Successful" }
				failure{ echo "Deploy to CLOUDHUB - Failure" }
			}
		}
		
		stage("Run unit tests"){
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

