#!groovy

SONAR_BRANCH='master'
SONAR_URL='http://10.127.126.113:9000'
SonarQubeEnv ='Sonarqube_scanner_lutron_sharepoint'

pipeline {
    agent {  
            node {
               label "Windows_Slave"
               customWorkspace "C:/jenkins/workspace/Build"
                 }
	         
	     }
		 
		 environment {

        EMAIL_RECIPIENTS = 'ashutosh.tiwari01@nagarro.com'
		}
		 
    stages {
	
		
        stage('Build & SonarScanner'){
		     steps{
		            
                echo 'Build'
                SonarAnalysis()				
                bat "\"${tool 'MSBuild'}\\msbuild.exe\" mvcToDoList/mvcToDoList.sln /p:Configuration=Debug /p:Platform=\"Any CPU\" /p:VisualStudioVersion=15.0 /p:ProductVersion=1.0.0.${env.BUILD_NUMBER} /v:diag"
                bat "\"${tool 'MSBuild'}\\msbuild.exe\" /t:Package mvcToDoList/mvcToDoList/mvcToDoList.csproj"
                bat "\"${tool 'sonarqube'}\\SonarScanner.MSBuild.exe\" end /d:sonar.login=6c0d6aae91d338cfcf6efacb0e0542b8b8603624"
              
            }
        }
		
		stage('Unit Testing') {
            steps {
                echo 'Unit Test Running'
				//bat 'karma start "AUT Jasmine"/karmaconf.js'
             }
			}
        
		
		
		
		stage('nuget packaging') {
            steps {
                echo 'nuget Packaging'
								bat 'C:/jenkins/nuget.exe pack mvcToDoList/mvcToDoList'
            }
        }
		
		stage('nuget publish & install') {
            steps {
                echo 'Nuget publish & install'
				//bat 'C:/jenkins/nuget.exe setapikey admin:AP4WwMhAP8FPbyppRfqqckxtXXZ -Source Artifactory'
                bat 'C:/jenkins/nuget.exe push *.nupkg -source "C:/jenkins/Todo/Packages"'
                bat 'C:/jenkins/nuget.exe install Viacom.Intranet.Asia -source "C:/jenkins/Todo/Packages" -OutputDirectory "C:/jenkins/Todo/Packages/Install"'
				            }
        }
		
		stage('Deploy to Acceptance') {
		
		when {

                // check if branch is master

                branch 'master'

            }

            steps {

                script {

                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') 

                        timeout(time: 3, unit: 'MINUTES') {

                            //input message:'Approve deployment?', submitter: 'Ashutosh Tiwari'

                            input message: 'Approve deployment?'

                        }
                                             

                             else {

                                error 'the application is not deployed !'

                            }

                        }

                    }

                }

                    
		
	//	stage('deploying artifact') {
    //        steps {
      //          echo 'Deployment Started'
				//powershell './SPdeploymentscript.ps1'
           // }
       // }
		
			
		
    }
	 post {

        // Always runs. And it runs before any of the other post conditions.

         success {

            sendEmail("Successful");

        }

        unstable {

            sendEmail("Unstable");

        }

        failure {

            sendEmail("Failed");

       }
    }
}

 
	
	def SonarAnalysis()
{
 echo  "\u2600 **********Sonar analysis started*****************"
withSonarQubeEnv("$SonarQubeEnv") {
    bat "\"${tool 'sonarqube'}\\SonarScanner.MSBuild.exe\" begin /key:DevOps_1234 /name:Todo /v:1.0 /d:sonar.host.url=http://10.127.126.113:9000 /d:sonar.login=6c0d6aae91d338cfcf6efacb0e0542b8b8603624"
    }
}


def sendEmail(status) {

      mail(

            to: "$EMAIL_RECIPIENTS",

            subject: "Jenkins Job $BUILD_NUMBER - " + status + " ToDo",

            body: "Changes:\n " + getChangeString() + "\n\n Check Jenkins Console output at: $BUILD_URL/console" + "\n" + "\n\n Check Sonarqube output at: http://10.127.126.113:9000/dashboard/index/todo" + "\n")

}

def getChangeString() {

    MAX_MSG_LEN = 100

    def changeString = ""



    echo "Gathering SCM changes"

    def changeLogSets = currentBuild.changeSets

    for (int i = 0; i < changeLogSets.size(); i++) {

        def entries = changeLogSets[i].items

        for (int j = 0; j < entries.length; j++) {

            def entry = entries[j]

            truncated_msg = entry.msg.take(MAX_MSG_LEN)

            changeString += " - ${truncated_msg} [${entry.author}]\n"

        }

    }



    if (!changeString) {

        changeString = " - No new changes"

    }

    return changeString

}

