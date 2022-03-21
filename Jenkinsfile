pipeline {
    agent any
     
	environment {
		MULE_CRED = credentials('mule.credentials')
		NEXUS_CRED = credentials('nexus.credentials')
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
        NEXUS_URL = "localhost:9081"
        NEXUS_REPOSITORY = "maven-snapshots"
                
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
	    stage('deploy-to-nexus') {
		    when { expression { env.BRANCH_NAME == "develop"} }
			steps{
				script {
				try{
					// use profile nexus (-P nexus) to deploy to Nexus.
        				bat "mvn clean deploy -P nexus"
					currentBuild.result = 'SUCCESS'
				}
				catch(Exception e){
					currentBuild.result = 'FAILURE'
				}
				}
			}      		
    		}
		stage('Publish to Exchange'){
			when { expression { env.BRANCH_NAME == "develop"} }
			steps{
				script {
				try{
					bat 'mvn clean deploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -s mvn-settings.xml'		
					currentBuild.result = 'SUCCESS'
				}
				catch(Exception e){
					currentBuild.result = 'FAILURE'
				}
				}
			}
		}
		
		
		stage("Deploy to CLOUDHUB") {
            parallel {
                stage("DEV") {
                    when { expression { env.BRANCH_NAME == "develop" } }
                    steps {
						script {
						try{
							bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=dev'
							currentBuild.result = 'SUCCESS'
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
						}
						}
                    }
                }
                stage("TEST") {
                    when { expression { env.BRANCH_NAME == "release" && env.TARGET_ENV == "test" } }
                    steps {
					    script {
						try{
							bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=test'
							currentBuild.result = 'SUCCESS'
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
						}
						}
                    }
                }
                stage("STAGE") {
                    when { expression { env.BRANCH_NAME == "release" && env.TARGET_ENV == "stage" } }
                    steps {
					    script {
						try{
                        	bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=stage'
							currentBuild.result = 'SUCCESS'
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
						}
						}
                    }
                }
				stage("CERT") {
                    when { expression { env.BRANCH_NAME == "release" && env.TARGET_ENV == "cert" } }
                    steps {
					    script {
						try{
							bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=cert'
							currentBuild.result = 'SUCCESS'
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
						}
						}
                    }
                }
				stage("PROD") {
                    when { expression { env.BRANCH_NAME == "master"} }
                    steps {
					    script {
						try{
							bat 'mvn clean deploy -DmuleDeploy -Dusername=${MULE_CRED_USR} -Dpassword=${MULE_CRED_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=prod'
							currentBuild.result = 'SUCCESS'
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
						}
						}
                    }
                }
            }
        }
	    


		stage("Perform Integration Test") {
            parallel {
                stage("TEST") {
                    when { expression { env.BRANCH_NAME == "release" && env.TARGET_ENV == "test" } }
                    steps {
						script {
						try{
							bat 'npm run app-tests-test'
							currentBuild.result = 'SUCCESS'
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
						}
						junit 'newman.xml'
                    }
					}
                }
                stage("STAGE") {
                    when { expression { env.BRANCH_NAME == "release" && env.TARGET_ENV == "stage" } }
                    steps {
					    script {
                        try{
							bat 'npm run app-tests-stage'
							currentBuild.result = 'SUCCESS'
						}
						catch(Exception e){
							currentBuild.result = 'FAILURE'
						}
						junit 'newman.xml'
                    }
					}
                }
                stage("CERT") {
                    when { expression { env.BRANCH_NAME == "release" && env.TARGET_ENV == "cert" } }
                    steps {
					    script {
                        try{
							bat 'npm run app-tests-cert'
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
