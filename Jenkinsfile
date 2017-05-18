node('maven') {
    stage('Prepare') {
        git url:
'http://jenkins:jenkins@gogs-user2-gogs.apps.advdev.openshift.opentlc.com/rlichti/kitchensink.git'

        sh 'mvn -s .openshift/nexus_settings.xml -P openshift build-helper:parse-version versions:set -DbuildNumber=$BUILD_NUMBER \'-DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}-${buildNumber}\''
        def pom = readMavenPom file: 'pom.xml'
        
        
        
        sh 'git checkout -b build-' + pom.version
        sh 'git config user.email jenkins@example.opentlc.com && git config user.name jenkins'
        sh 'git push origin build-' + pom.version
    }
    
    stage('Build') {
        sh "mvn -s .openshift/nexus_settings.xml -P openshift -DskipTests=true -DskipITs=true -DskipUTs=true clean package"
    }

    stage('Tests') {
//        sh "mvn -s .openshift/nexus_settings.xml -P openshift test"
//        junit '**/target/surefire-reports/TEST-*.xml'
    }
    
    stage('Code Quality Tests') {
//        sh "mvn -s .openshift/nexus_settings.xml -P openshift -DskipTests=true
//        -DskipITs=true -DskipUTs=true
//        -Dsonar.host.url=http://sonarqube-user2-sonarqube.apps.advdev.openshift.opentlc.com
//        sonar:sonar"
    }
    
    stage('Publish') {
        archive 'target/*.war'
        sh "mvn -s .openshift/nexus_settings.xml -P openshift -DskipTests=true -DskipITs=true -DskipUTs=true -DaltDeploymentRepository=nexus::default::http://nexus3-user2-nexus.apps.advdev.openshift.opentlc.com/repository/maven-releases deploy"
    }
    
    stage('Build OpenShift Image') {
    }
    
    stage('Deploy Green/Blue') {
        
    }
    
    input 'Activate Version?'
    
    stage('Switch Green/Blue') {
    }
}
