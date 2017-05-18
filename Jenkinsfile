node {
    sh 'echo === Environment of host ==='
		sh 'hostname'
		sh 'env'
    sh 'echo ==========================='
}

node('maven') {
    sh 'echo === Environment of host ==='
		sh 'hostname'
		sh 'env'
    sh 'echo ==========================='

		def gitPipeline = 'http://jenkins:jenkins@gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink-pipeline.git'
		def gitSource = 'http://jenkins:jenkins@gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink.git'

		def mvnCommand = 'mvn -s .openshift/nexus_settings.xml -P openshift '
		def mvnNonTest = mvnCommand + '-DskipTests=true -DskipITs=true -DskipUTs=true '

		def prepareGitPush = 'git config user.email jenkins@example.opentlc.com && git config user.name jenkins'
		def nexusUrl = 'http://nexus3-user2-nexus.apps.advdev.openshift.opentlc.com/repository/maven-releases'

    stage('Prepare') {
        git url: gitSource

        sh mvnNonTest + 'build-helper:parse-version versions:set -DbuildNumber=$BUILD_NUMBER \'-DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}-${buildNumber}\''

        def pom = readMavenPom file: 'pom.xml'
        def branchName = 'build-' + pom.version

        sh 'git checkout -b ' + branchName
				sh prepareGitPush
        sh 'git push origin ' + branchName
    }
    
    stage('Build') {
        sh mvnNonTest + 'clean package'
    }

    stage('Tests') {
//        sh $mvnCommand + 'test'
//        junit '**/target/surefire-reports/TEST-*.xml'
    }
    
    stage('Code Quality Tests') {
//        sh mvnNonTest + '-Dsonar.host.url=http://sonarqube-user2-sonarqube.apps.advdev.openshift.opentlc.com sonar:sonar"
    }
    
    stage('Publish WAR') {
        archive 'target/*.war'
        sh mvnNonTest + "-DaltDeploymentRepository=nexus::default::$nexusUrl deploy"

        def pom = readMavenPom file: 'pom.xml'
				def branchName = 'build-' + pom.version

				sh 'git ' + gitPipeline + ' pipeline'
				sh 'cd pipeline'
				sh 'git checkout -b ' + branchName

				// first time using only > to overwrite the current file
				sh 'echo NEXUS_HOST="$nexusUrl" 																 >  .s2i/environment'
				sh 'echo WAR_VERSION="'			+ pom.version 									+ '" >> .s2i/environment'
				sh 'echo MAVEN_GROUP="'			+ pom.groupId.replace('.','/') 	+	'" >> .s2i/environment'
				sh 'echo MAVEN_ARTIFACT="' 	+ pom.artifactId                + '" >> .s2i/environment'
				sh 'echo WAR_FILE_LOCATION="' + nexusUrl
																			+ '/' + pom.groupId.replace(".","/") 
																			+ '/' + pom.version
																			+ '/' + pom.artifactId 
																			+ '-' + pom.version
																			+ '.war" >> .s2i/environment'

				sh 'git commit -am "Build run ' + branchName + '"'
				sh prepareGitPush
        sh 'git push origin ' + branchName
    }
    
    stage('Build OpenShift Image') {
				
    }
    
    stage('Publish Green/Blue') {
        
    }
    
		stage('Go Live') {
			def pom = readMavenPom file: 'pom.xml'
			def branchName = 'build-' + pom.version

    	input 'Activate Version ' + branchName + '?'
		} 
    
    stage('Switch Green/Blue') {
    }
}
