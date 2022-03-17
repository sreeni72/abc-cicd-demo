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
    }
}
	

