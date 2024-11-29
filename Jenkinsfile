def registry = 'https://jinendra.jfrog.io'

pipeline{
	agent any
	environment {
	    PATH= "/opt/maven/bin:$PATH"	
	}
	stages{
	      stage("build"){	
	        steps {
		  echo "-------- build started -------------"
	          sh 'mvn clean deploy -Dmaven.test.skip=true'
		  echo "--------build ended ----------------"
	         }
	       }
	  	stage("test"){
                steps {
		 echo "-------- unit test started -------------"
                  sh 'mvn surefire-report:report'
		 echo "-------- unit test ended -------------"
                 }
               }

	       stage("SonarQube analysis"){ 
		    environment{
			scannerHome = tool 'jinu-sonar-scanner'
		    }
		    steps {
			withSonarQubeEnv('jinu-sonarqube-server'){
				sh "${scannerHome}/bin/sonar-scanner"
			  } 
			}
	}
	

		stage("Quality Gate") {               // 11  // Creates a stage named 'Quality Gate'
            steps {                           // 12  // Defines the steps that will be executed in this stage
                script {                      // 13  // Allows running custom Groovy script inside the pipeline
                    timeout(time: 1, unit: 'HOURS') {  
                                              // Sets a timeout of 1 hour for the quality gate check
                        def qg = waitForQualityGate()  
                                              // Waits for the quality gate result from SonarQube
                        if (qg.status != 'OK') {  
                                              // Checks if the quality gate status is not OK
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"  
                                              // Aborts the pipeline if the quality gate fails
                        }
                    }
                }                             // 13  // Ends the script block for the Quality Gate stage
            }                                 // 12  // Ends the steps block for 'Quality Gate' stage
        }                                     // 11  // Ends the 'Quality Gate' stage

        stage("Jar Publish") {                // 14  // Creates a stage named 'Jar Publish'
            steps {                           // 15  // Defines the steps that will be executed in this stage
                script {                      // 16  // Allows running custom Groovy script inside the pipeline
                    echo '<--------------- Jar Publish Started --------------->'  
                                              // Logs a message indicating the start of JAR publishing
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifact-cred"  
                                              // Defines the Artifactory server with the specified URL and credentials
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"  
                                              // Sets properties like build ID and Git commit ID for the build
                    def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "jinu_artifactory-libs-release-local/{1}",
                              "flat": "false",
                              "props": "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""  
                                              // Defines the upload specification for uploading JAR files to Artifactory
                    def buildInfo = server.upload(uploadSpec)  
                                              // Uploads the files to Artifactory and collects build info
                    buildInfo.env.collect()  
                                              // Collects environment variables as part of the build info
                    server.publishBuildInfo(buildInfo)  
                                              // Publishes the build information to Artifactory
                    echo '<--------------- Jar Publish Ended --------------->'  
                                              // Logs a message indicating the end of JAR publishing
                }                             // 16  // Ends the script block for 'Jar Publish' stage
            }                                 // 15  // Ends the steps block for 'Jar Publish' stage
        }                                     // 14  // Ends the 'Jar Publish' stage

    }                                         // 3  // Ends the stages block
}                                             // 1  // Ends the pipeline block		

