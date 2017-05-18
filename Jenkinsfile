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

        sh mvnNonTest + 'build-helper:parse-version versions:set -DbuildNumber=$BUILD_NUMBER \'-DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}-${buildNumber}\''

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

        sh 'git clone ' + gitOCP + ' ocp'
			  sh '(cd ocp && ' + prepareGitPush + ')'
        sh '(cd ocp && git checkout -b ' + branchName + ')'

        // first time using only > to overwrite the current file
        sh 'echo NEXUS_HOST="'      + nexusUrl                      + '"  > ocp/.s2i/environment'
        sh 'echo WAR_VERSION="'     + pom.version                   + '" >> ocp/.s2i/environment'
        sh 'echo MAVEN_GROUP="'     + pom.groupId.replace('.','/')  + '" >> ocp/.s2i/environment'
        sh 'echo MAVEN_ARTIFACT="'  + pom.artifactId                + '" >> ocp/.s2i/environment'
        sh 'echo WAR_FILE_LOCATION="' + nexusUrl + '/' + pom.groupId.replace('.','/') + '/' + pom.artifactId + '/' + pom.version + '/' + pom.artifactId + '-' + pom.version + '.war" >> ocp/.s2i/environment'

				sh 'sed -e \'s|###GITREPO###|' + gitOCPBranched + '|g\' ocp/bc-kitchensink-imagecreator.template > ocp/bc-kitchensink-imagecreator.yaml'
				sh '(cd ocp && git add bc-kitchensink-imagecreator.yaml)'

        sh '(cd ocp && git commit -am "Build run ' + branchName + '")'
        sh '(cd ocp && git push origin ' + branchName + ')'
    }
}

node { 
    stage('Build OpenShift Image') {
				sh 'git clone ' + gitOCP + '#' + branchName + ' ocp'
				sh 'oc create -f ocp/bc-kitchensink-imagecreator.yaml'

				openshiftBuild(buildConfig: 'kitchensink-imagecreator', showBuildLogs: 'true')
				sh 'oc delete bc/kitchensink-imagecreator'
    }
    
    stage('Publish Green/Blue') {
        
    }
    
		stage('Go Live') {
    	input 'Activate Version ' + branchName + '?'
		} 
    
    stage('Switch Green/Blue') {
    }
}
