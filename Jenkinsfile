node {
		stage('Playground') {
		}
}

node('maven') {
		def gitOCP = 'http://jenkins:jenkins@gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink-ocp.git'
		def gitSource = 'http://jenkins:jenkins@gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink.git'

		def nexusUrl = 'http://nexus3-user2-nexus.apps.advdev.openshift.opentlc.com/repository/maven-releases'

		def mvnCommand = 'mvn -s .openshift/nexus_settings.xml -P openshift '
		def mvnNonTest = mvnCommand + '-DskipTests=true -DskipITs=true -DskipUTs=true '
		def prepareGitPush = 'git config user.email jenkins@example.opentlc.com && git config user.name jenkins'

		def branchName = env.BUILD_TAG
		def gitOCPBranched = gitOCP + '#' + branchName

    stage('Prepare') {
        git url: gitSource
        sh 'git checkout -b ' + branchName

        sh mvnNonTest + 'build-helper:parse-version versions:set -DbuildNumber=$BUILD_NUMBER \'-DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}-${buildNumber}\''

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

				git url: gitOCP

        // first time using only > to overwrite the current file
        sh 'echo NEXUS_HOST="'      + nexusUrl                      + '"  > .s2i/environment'
        sh 'echo WAR_VERSION="'     + pom.version                   + '" >> .s2i/environment'
        sh 'echo MAVEN_GROUP="'     + pom.groupId.replace('.','/')  + '" >> .s2i/environment'
        sh 'echo MAVEN_ARTIFACT="'  + pom.artifactId                + '" >> .s2i/environment'
        sh 'echo WAR_FILE_LOCATION="' + nexusUrl + '/' + pom.groupId.replace('.','/') + '/' + pom.artifactId + '/' + pom.version + '/' + pom.artifactId + '-' + pom.version + '.war" >> .s2i/environment'

			  sh prepareGitPush
        sh 'git commit -am "Build run ' + branchName + '"'
        sh 'git tag ' + branchName
        sh 'git push origin'
    }
}

node { 
		def branchName = env.BUILD_TAG		

    stage('Build OpenShift Image') {
    }
    
    stage('Publish Green/Blue') {
        
    }
    
		stage('Go Live') {
    	input 'Activate Version ' + branchName + '?'
		} 
    
    stage('Switch Green/Blue') {
    }
}
