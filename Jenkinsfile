DEVemailid = [ 'linuxsbk4@gmail.com', 'linuxsbk@gmail.com', 'bsivasub@redhat.com' ]
QAemailid = "linuxsbk4@gmail.com"
UATemailid= "linuxsbk4@gmail.com"
SDEmailId = "linuxsbk4@gmail.com"

waitingTime = 24

repositoryName = "testing_pipeline"


// nugetpath = tool 'nuget-default'

def nugetpath

/* def notifydev(String buildStatus = 'STARTED') {
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

*/
pipeline{
    agent any
   
    
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
                    sh 'echo "Restoring Dependencies" > 1restore_dependency'             
                        }}}

    stage('Build') {
        steps{
            script{
                sh 'echo Build > 2build'
            }}}
	    
      
    stage ('Nunit test') {
         steps{
            script{
                sh 'echo "Nunit test" > 3nunit'
	    }}}
	    
	stage('SonarQube analysis') {
        steps{
				script {
                    sh 'echo "SonarQube analysis" > 4sonar'
			}}} 

}

    
post { 
  success { 
  	    mail to: "${DEVemailid}",
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
node {
	stage ('Promote Artifacts to QA'){
        sh 'echo "Promote Artifacts to QA" > 5promoteqa'
		    } 

stage ('Approve to Proceed'){
notifyQA()
proceedConfirmation("QAtoUAT","Promote to UAT ?")
}
//agent any
stage ('Promote Artifacts to UAT'){
	sh 'echo "Promote Artifacts to UAT" > 6promoteUAT'
	 }

stage ('Approve to Proceed'){
notifyUAT()
proceedConfirmation("UATtoProd","Promote to Prod ?")
}
//agent any
stage ('Promte Artifacts to Prod'){
    sh 'echo "Promote Artifcats to Prod" > 7promotePROD'
 }

notifySD()
} 
		


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


 /*
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
}


*/
   
	
