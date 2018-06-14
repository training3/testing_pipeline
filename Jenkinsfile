DEVemailid = "linuxsbk4@gmail.com"
QAemailid = "linuxsbk4@gmail.com"
UATemailid= "linuxsbk4@gmail.com"

waitingTime = 24

repositoryName = "WAMGateway"


// nugetpath = tool 'nuget-default'

def nugetpath
pipeline{
    agent { node 'win' }
   
    
stages { 
    stage('Clone') {
    steps {
        script {
            checkout scm 
	    echo "${repositoryName}"
                    }
    }
    }

    stage('Restore Dependency'){
        steps{
            script{
		dir('WAMBuild') {
                    bat '''rem ensure the registry setting is present
                        set "var=%~dp0"
                        regedit.exe -s %var%msbuild-enable-out-of-proc.reg

                        rem removing the \\packages folder (if present).
                        rmdir ..\\packages /s /y

                        rem restore nuget
                        dir
			nuget.exe restore ..\\wamgateway.sln
                        '''
             
                        }}}}

    stage('Build') {
        steps{
            script{
    dir('WAMBuild') {

            bat "\"${tool 'msbuild-default'}/msbuild\" wam.msbuild /t:Clean,Build /p:Configuration=Desktop"

            bat "\"${tool 'msbuild-default'}/msbuild\" wam.msbuild /t:BuildSetup /p:Configuration=Desktop"

            bat "\"${tool 'msbuild-default'}/msbuild\" wam.msbuild /t:Clean,Build /p:Configuration=Citrix"

            // rem Call the MSBuild project for Citrix SETUP target..
            bat "\"${tool 'msbuild-default'}/msbuild\" wam.msbuild /t:CitrixBuildSetup /p:Configuration=Citrix"

          }}}}
	    
      
    stage ('Nunit test') {
         steps{
            script{
     dir('WAMBuild') {
	    
	    // rem Build the NUnit test project
            bat "\"${tool 'msbuild-default'}/msbuild\" wam.msbuild /t:Clean,BuildTestProject /p:Configuration=Release"

            // echo Build completed.


            // :End
           // '''

	   }
           
	    }}}
	    
	stage('SonarQube analysis') {
        steps{
				script {
		        		def scannerHome = tool 'SonarQube';
                                         withSonarQubeEnv('sonarQube') {                                       
					bat "${scannerHome}/bin/sonar-scanner"

			}}
		 }} 


    stage('Upload artifacts') {
        steps{
			  script {
				def server = Artifactory.server ('artifacts')
				def uploadSpec = """{
				"files": [
				   {
				   "pattern": "WAMGateway-Citrix/Citrix/*.msi",
				   "target": "nuget-local/WAMGateway/dev/${env.BUILD_NUMBER}/"
				   },
				   {
				   "pattern": "WAMGatewaySetup/Desktop/*.msi",
				   "target": "nuget-local/WAMGateway/dev/${env.BUILD_NUMBER}/"
				}
				         ]

			        }"""

				def buildInfo1 = server.upload(uploadSpec)

				server.publishBuildInfo(buildInfo1)

			       }}}
}

    
post { 
  success { 
  					mail to: 'linuxsbk4@gmail.com',
            subject:  "Suceeded:  ${currentBuild.fullDisplayName}", 
            body: "Build succeeded ${env.BUILD_URL} ${env.JOB_NAME} ${env.BUILD_NUMBER}"   
  				}
  failure {
    				mail to: 'linuxsbk4@gmail.com',
            subject: "Failed: ${currentBuild.fullDisplayName}",
            body: "Build failed ${env.BUILD_URL} ${env.JOB_NAME} ${env.BUILD_NUMBER}"
  				} }}




 // Stage: promote
stage ('Approve to Proceed'){	
notifydev()
	 proceedConfirmation("proceed1","promote to QA?")
}
node('WCW32983') {
	stage ('Promote Artifacts to QA'){
def server = Artifactory.server 'artifacts'

def uploadSpec  =  """{
"files": [
 {
    "pattern": "WAMGateway-citrix/citrix/*.msi",
    "target": "nuget-local/WAMGateway/QA/${env.BUILD_NUMBER}/"
 },


{


                                "pattern": "WAMgatewaySetup/Desktop/*.msi",

                                 "target": "nuget-local/WAMGateway/QA/${env.BUILD_NUMBER}/"


                                }





]
}"""
def buildInfo1 = server.upload(uploadSpec)
				server.publishBuildInfo(buildInfo1)
			        }
		    } 


stage ('Approve to Proceed'){
notifyQA()
proceedConfirmation("QAtoUAT","Promote to UAT ?")
}
agent { node 'win' }
stage ('Promte Artifacts to UAT'){
	def server = Artifactory.server 'artifacts'
	def uploadSpec  =  """{

"files": [

 {

    "pattern": "WAMGateway-citrix/citrix/*.msi",

    "target": "nuget-local/WAMGateway/UAT/${env.BUILD_NUMBER}/"

 },

{


                                "pattern": "WAMGatewaySetup/Desktop/*.msi",

                                 "target": "nuget-local/WAMGateway/UAT/${env.BUILD_NUMBER}/"


                                }



]

}"""

	def buildInfo1 = server.upload(uploadSpec)

	server.publishBuildInfo(buildInfo1)

	 }


//} 

stage ('Approve to Proceed'){
notifyUAT()
proceedConfirmation("UATtoProd","Promote to Prod ?")
}
agent { node "win" }
stage ('Promte Artifacts to Prod'){
	def server = Artifactory.server 'artifacts'
	def uploadSpec  =  """{

"files": [

 {

    "pattern": "WAMGateway-citrix/citrix/*.msi",

    "target": "nuget-local/WAMGateway/Prod/${env.BUILD_NUMBER}/"

 },

{


                                "pattern": "WAMGatewaySetup/Desktop/*.msi",

                                 "target": "nuget-local/WAMGateway/Prod/${env.BUILD_NUMBER}/"


                                }



]

}"""

	def buildInfo1 = server.upload(uploadSpec)

	server.publishBuildInfo(buildInfo1)

	 }

notifySD()
//} 
		


 def notifydev(String buildStatus = 'STARTED') {
	// build status of null means successful
	buildStatus =  buildStatus ?: 'SUCCESSFUL'
	def toList = DEVemailid
	def subject = "DEV: '${repositoryName}' artifact ready for promotion to QA"
	def summary = "${subject} (${env.BUILD_URL})"
	def details = """
	<p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' is ready to be promoted from dev to qa.</p>
	<p>Click here to move the library into the QA artifactory for testing. "<a href="${env.BUILD_URL}/input">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>
	"""

	emailext body: details,mimeType: 'text/html', subject: subject, to: toList
	}

 def notifyQA(String buildStatus = 'SUCCESSFUL') {

	// build status of null means successful

	buildStatus =  buildStatus ?: 'SUCCESSFUL'
	def toList = QAemailid
	def subject = "QA: '${repositoryName}' artifact Moved to UAT for Test"
	def summary = "${subject} (${env.BUILD_URL})"
	def details = """

	<p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' tested successfull in QA and pushed to UAT for Testing.</p>
	<p> Hello UAT Team,</p>
	<p> QA team has published artifacts to UAT repository , Please perform Test againest new build </p> 
	<p> Based on results click on the below link to promte artifacts to Release artifactory or Abort </p>
        <p> Build approval link. "<a href="${env.BUILD_URL}/input">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>

	"""

	emailext body: details,mimeType: 'text/html', subject: subject, to: toList

	}


 def notifyUAT(String buildStatus = 'SUCCESSFUL') {

	// build status of null means successful

	buildStatus =  buildStatus ?: 'SUCCESSFUL'
	def toList = UATemailid
	def subject = "UAT: '${repositoryName}' artifact Moved to Prod "
	def summary = "${subject} (${env.BUILD_URL})"
	def details = """

	<p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' tested successfull in UAT and pushed to Prod .</p>
	<p> Hello UAT Team,</p>
	<p> UAT team has published artifacts to Prod repository  </p> 
	<p> Based on results click on the below link to promte artifacts to Release artifactory or Abort </p>
        <p> Build approval link. "<a href="${env.BUILD_URL}/input">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>

	"""

	emailext body: details,mimeType: 'text/html', subject: subject, to: toList

	}
 def notifySD(String buildStatus = 'SUCCESSFUL') {
	// build status of null means successful
	buildStatus =  buildStatus ?: 'SUCCESSFUL'
	def toList = SDEmailId
	def subject = "SD: '${repositoryName}' artifact ready for promotion to Prod"
	def summary = "${subject} (${env.BUILD_URL})"
	def details = """
	<p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' is ready to Deploy to Prod.</p>
	"""

	emailext body: details,mimeType: 'text/html', subject: subject, to: toList
	}




    def proceedConfirmation(String id, String message) {	
    def userInput = true
    def didTimeout = false
    try {
        timeout(time: waitingTime, unit: 'HOURS') { //
        userInput = input(
          id: "${id}", message: "${message}", parameters: [
          [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Confirm to proceed !']
          ])
      }
    } catch(e) { // timeout reached or input false
        def user = e.getCauses()[0].getUser()
        if('SYSTEM' == user.toString()) { // SYSTEM means timeout.
         didTimeout = true
         if (didTimeout) {
            echo "no input was received abefore timeout"
            currentBuild.result = "FAILURE"
            throw e
            } else if (userInput == true) {
                echo "this was successful"
            } else {
                userInput = false
                echo "this was not successful"
                currentBuild.result = "FAILURE"
                println("catch exeption. currentBuild.result: ${currentBuild.result}")
                throw e
            }
       } else {
         userInput = false
         echo "Aborted by: [${user}]"
     }
  }
   
 }

   
	
