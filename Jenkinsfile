#!groovy

def mvn         	= 'mvn -s .openshift/nexus_settings.xml -P openshift'
def skipTests     = '-DskipTests=true -DskipITs=true -DskipUTs=true'

def prepareGitPush = "git config user.email jenkins@example.opentlc.com && git config user.name jenkins"

def sonarqube     = "http://sonarqube-user2-sonarqube.apps.advdev.openshift.opentlc.com"

def credentialsId = '180c9d3f-b378-4b31-823f-386634cb906e'
def gitSource     = 'http://gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink.git'
def ocpRepo       = 'http://gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink-ocp.git'

def nexusUrl      = 'http://nexus3-user2-nexus.apps.advdev.openshift.opentlc.com/repository/maven-releases'

def branchName    = env.BUILD_TAG

def application   = "kitchensink"
def routeGreen		= "${application}-green"
def routeBlue			= "${application}-blue"

def testEnv       = "dev-${env.BUILD_TAG}"
def prodEnv      	= "user2-kitchensink-prod"

node('maven') {
  stage('SCM') {
    checkout scm

    git url: gitSource
  }

  def pom = readMavenPom file: 'pom.xml'

  stage('Prepare') {
    sh "git checkout -b ${env.BUILD_TAG}"

    sh mvn + " " + skipTests + ' build-helper:parse-version versions:set -DbuildNumber=' + env.BUILD_TAG + ' \'-DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}-${buildNumber}\''

    sh prepareGitPush
    sh "git push origin ${env.BUILD_TAG}"
  }
    
  stage('Build') {
    sh "${mvn} ${skipTests} clean package"
  }

  stage('Tests') {
//    sh "${mvn} test"
//    junit '**/target/surefire-reports/TEST-*.xml'
  }
    
  stage('Code Quality Tests') {
//    sh "${mvn} ${skipTests} -Dsonar.host.url=${sonarqube} sonar:sonar"
  }
    
  stage('Publish WAR') {
    sh "${mvn} ${skipTests} -DaltDeploymentRepository=nexus::default::$nexusUrl deploy"

    git url: gitOCP

    sh "echo WAR_FILE_LOCATION=\"${nexusUrl}/${pom.groupId.replace('.','/')}/${pom.artifactId}/${pom.version}/${pom.artifactId}-${pom.version}.war\" > .s2i/environment"

    sh prepareGitPush
    sh "git commit -am \"Build run ${env.BUILD_TAG}\""
    sh "git tag ${env.BUILD_TAG}"
    sh "git push origin"
  }

  stage('Prepare Test Environment') {
    echo "Creating test environment: ${testEnv} ..."
    sh "oc new-project ${testEnv}"
  }
  
  stage('Build OpenShift Image') {
    echo "Build the image ..."
    openshiftBuild  bldCfg: 'kitchensink-imagecreator', 
                    showBuildLogs: 'true',
                    namespace: testEnv, 
                    checkForTriggeredDeployments: 'false',
                    verbose: 'false'

    openshiftTag    destTag: env.BUILD_TAG,
                    destStream: 'kitchensink',
                    destinationNamespace: testEnv,
                    srcStream: 'kitchensink',
                    srcTag: 'latest',
                    namespace: testEnv,
                    alias: 'false',
                    verbose: 'false'
  }

  stage('Deploy to Testing Environment') {
    echo "Deploy to Testing Environment ..."

    openshiftTag    destTag: 'rft',
                    destStream: 'kitchensink',
                    destinationNamespace: testEnv,
                    srcStream: 'kitchensink',
                    srcTag: env.BUILD_TAG,
                    namespace: testEnv,
                    alias: 'false',
                    verbose: 'false'

    openshiftDeploy deploymentConfig: 'kitchensink',
                    namespace: testEnv
  }


  stage('Publish Green/Blue') {
    sh "oc project ${prodEnv}"
    sh "oc get route ${application} -n ${prodEnv} -o template --template='{{ .spec.to.name }}' > route-target"

		def current = currentTarget()
		echo "Deploy to production: ${current}"

    openshiftTag    destTag: "prod",
                    destStream: "kitchensink",
                    destinationNamespace: testEnv,
                    srcStream: 'kitchensink',
                    srcTag: env.BUILD_TAG,
                    namespace: testEnv,
                    alias: 'false',
                    verbose: 'false'

  }
    
  stage('Go Live') {
    def current = currentTarget();
		
		if (current != "") {
    	def switchTo = newTarget()

    	input "Activate version '${env.BUILD_TAG}' by switching from '${current}' to '${switchTo}' in environment '${prodEnv}'?"

    	sh "oc patch -n ${prodEnv} route/${application} --patch '{\"spec\":{\"to\":{\"name\":\"${switchTo}\"}}}'"
		}
  }

  stage('Tear Down Testing Environment') {
    echo "Tearing down testing environment: ${testEnv} ..."
    sh "oc delete project ${testEnv}"
  }

}

def currentTarget() {
  def result = readFile 'route-target'
  return result
}

def newTarget() {
  def current = currentTarget()
  def result = ""

  if (current == routeBlue) {
    result = routeGreen
  } else if (current == routeGreen) {
    result = routeBlue
  } else {
    echo "Sorry, route '${current}' not known. Can't generate new target!"
  }

  return result
}
