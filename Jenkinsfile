pipeline {
    agent any
       
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
		environment {
			ANYPOINT_CREDENTIALS = credentials('anypoint.credentials')
		}
		steps{
			echo "Publishing to Exchange Starting....."
			bat 'mvn clean deploy -Dusername=${ANYPOINT_CREDENTIALS_USR} -Dpassword=${ANYPOINT_CREDENTIALS_PSW} -s settings.xml'			
			echo "Publishing to Exchange Completed....."
		}
	}
	stage('Deploy to CLOUDHUB'){
		environment {
			ANYPOINT_CREDENTIALS = credentials('anypoint.credentials')
		}
		steps{
			echo "Deploying to CLOUDHUB Starting....."
			bat 'mvn clean deploy -DmuleDeploy -Dusername=${ANYPOINT_CREDENTIALS_USR} -Dpassword=${ANYPOINT_CREDENTIALS_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=dev -Dmule.version=4.4.0'
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
