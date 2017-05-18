node('maven') {
		def gitPipeline = 'http://jenkins:jenkins@gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink-pipeline.git'
		def gitSource = 'http://jenkins:jenkins@gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink.git'

		def nexusUrl = 'http://nexus3-user2-nexus.apps.advdev.openshift.opentlc.com/repository/maven-releases'

		def mvnCommand = 'mvn -s .openshift/nexus_settings.xml -P openshift '
		def mvnNonTest = mvnCommand + '-DskipTests=true -DskipITs=true -DskipUTs=true '
		def prepareGitPush = 'git config user.email jenkins@example.opentlc.com && git config user.name jenkins'

		def branchName = env.BUILD_TAG
		def gitPipelineBranched = gitPipeline + '#' + branchName

		stage('Playground') {
			git url: gitPipeline

			  sh prepareGitPush
        sh 'git checkout -b ' + branchName
				sh 'git push'

				sh 'git checkout -b master'

			  git url: gitPipelineBranched
		}

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

        sh 'git clone ' + gitPipeline + ' pipeline'
			  sh '(cd pipeline && ' + prepareGitPush + ')'
        sh '(cd pipeline && git checkout -b ' + branchName + ')'

        // first time using only > to overwrite the current file
        sh 'echo NEXUS_HOST="'      + nexusUrl                      + '"  > pipeline/.s2i/environment'
        sh 'echo WAR_VERSION="'     + pom.version                   + '" >> pipeline/.s2i/environment'
        sh 'echo MAVEN_GROUP="'     + pom.groupId.replace('.','/')  + '" >> pipeline/.s2i/environment'
        sh 'echo MAVEN_ARTIFACT="'  + pom.artifactId                + '" >> pipeline/.s2i/environment'
        sh 'echo WAR_FILE_LOCATION="' + nexusUrl + '/' + pom.groupId.replace('.','/') + '/' + pom.artifactId + '/' + pom.version + '/' + pom.artifactId + '-' + pom.version + '.war" >> pipeline/.s2i/environment'

				sh 'sed -e \'s|###GITREPO###|' + gitPipelineBranched + '|g' + pipeline/bc-kitchensink-imagecreator.template > pipeline/bc-kitchensink-imagecreator.yaml'
				sh '(cd pipeline && git add bc-kitchensink-imagecreator.yaml)'

        sh '(cd pipeline && git commit -am "Build run ' + branchName + '")'
        sh '(cd pipeline && git push origin ' + branchName + ')'
    }
}

node { 
    stage('Build OpenShift Image') {
				git url: gitPipelineBranched

				sh 'oc create -f bc-kitchensink-imagecreator.yaml'
				openshiftBuild(buildConfig: 'kitchensink-imagecreator', showBuildLogs:
				sh 'oc delete bc/kitchensink-imagecreator'
'true')
    }
    
    stage('Publish Green/Blue') {
        
    }
    
		stage('Go Live') {
    	input 'Activate Version ' + branchName + '?'
		} 
    
    stage('Switch Green/Blue') {
    }
}
