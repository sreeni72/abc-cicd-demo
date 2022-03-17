pipeline {
    agent any
       
    stages {
	stage('Build Application') { 
		steps {
			echo 'Build the Mule Application...' 
			bat 'mvn clean package'
		}
	}
	stage('Publish to Exchange'){
		steps{
			echo "Publishing to Exchange Starting....."
			withCredentials([usernamePassword(credentialsId: 'mule.credentials', passwordVariable: 'anypoint_pwd', usernameVariable: 'anypoint_user')]) {
    				bat 'mvn clean deploy -Dusername=${anypoint_user} -Dpassword=${anypoint_pwd} -s settings.xml'
    	    		}
			echo "Publishing to Exchange Completed....."
		}
	}
	stage('Deploy to CLOUDHUB'){
		steps{
			echo "Deploying to CLOUDHUB Starting....."
			withCredentials([usernamePassword(credentialsId: 'mule.credentials', passwordVariable: 'anypoint_pwd', usernameVariable: 'anypoint_user')]) {
    				bat 'mvn clean deploy -DmuleDeploy -Dusername=${anypoint_user} -Dpassword=${anypoint_pwd} -DworkerType=Micro -Dworkers=1 -Denvironment=dev -Dmule.version=4.4.0'
    	    		}
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
