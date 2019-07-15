import hudson.Util;
def label = "worker-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
  containerTemplate(name: 'gradle', image: 'gradle', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'tomcat', image: 'tomcat:8.0-alpine', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  ],
volumes: [
  hostPathVolume(mountPath: '/home/gradle/.gradle', hostPath: '/tmp/jenkins/.gradle'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
	try{
		node(label){
			stage('Initialize Workspace'){
				deleteDir()
			}
			gitrepo = checkout scm
			gitCommit = gitrepo.GIT_COMMIT
			gitBranch = gitrepo.GIT_BRANCH
			stage('Gradle Test'){
				try {
					container('gradle'){
						sh "/gradle/bin/gradle test"
					}
				}
				catch (exc) {
        			println "Failed to test - ${currentBuild.fullDisplayName}"
        		throw(exc)
     			}
     		}
     		stage('Gradle Build'){
     			try {
     			container('gradle'){
     				sh "/gradle/bin/gradle build"
     			}
     		} catch( exc ) {
     				error "Gradle Build Failed"
     				throw(exc)
     			}
     		}
     		stage('Docker Build'){
     			try {
     				container('docker'){
     					sh """
 							docker built -t sanjeev3d/tomcat-app:${gitCommit} .
 							"""
     				}	
     			}
     			catch( exc ) {
     				error "Docker Build Failed"
     				throw(exc)
     			}
     		}
     		stage('Docker Push'){
     			container('docker'){
     				withDockerRegistry(credentialsID: 'docker-hub-cred', url: 'https://hub.docker.com/'){
     					sh "docker push sanjeev3d/tomcat-app:${gitCommit}"
     				}
     			}
     		}
		}	
	} catch(exc){
		 currentBuild.result = 'FAILURE'
		throw(exc)
	}
}
