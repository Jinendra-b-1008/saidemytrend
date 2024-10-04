pipeline{
	agent any{
	environtment{
		PATH= "/opt/maven/bin:$PATH"
	}
	stages{
	   stage("build"){
	      steps{
	      sh 'mvn clean deploy' 
	    }
          }
	   stage("SonarQube analysis"){
	      environtment{
		scannerHome = tool 'jinendra-sonar-scanner'
	    }
	      steps{
		withSonarQubeEnv('jinendra-sonarqube-server'){
		  sh "${scannerHome}/bin/sonar-scanner"
	      }
	    }
	  }		
	}
}
