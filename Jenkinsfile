pipeline {
    
  agent {
      label 'worker-01'
  }

  parameters {
    string defaultValue: 'master', description: 'This is the branch to checkout the code', name: 'branch_name'
    choice choices: ['DEV', 'SIT', 'UAT'], description: 'Environment to be deployed', name: 'environment'
  }

  triggers {
    cron '00 20 * * *'
  }

  options {
    disableConcurrentBuilds()
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '20')
    timestamps()
  }
  
  tools {
    jdk 'java_home'
    maven 'maven3'
  }

  environment {
    data_path = "/opt/data/"
    SCANNER_HOME = tool 'sonarqube-scanner'
  }


stages {
  stage('Code Checkout') {
    steps {
      git branch: '$branch_name', credentialsId: 'github-credentials', url: 'https://github.com/gopishank/PetClinic.git'
    }
  }

  stage('Tests & Scans') {
      parallel {
          stage('Unit Testing'){
              steps {
                  sh "mvn test"
              }
          }
          stage('SonarQube Testing'){
              steps {
                  withSonarQubeEnv(installationName: 'sonarqube-server') {
                      sh "$SCANNER_HOME/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                  }
              }
          }
      }
  }
  
  stage('Package Code'){
      steps {
          sh "mvn package -Dmaven.test.skip=true"
      }
  }
  
  stage('Upload Artifacts'){
      steps {
          sh '''
          curl -u admin:adminadmin POST "http://15.207.222.196:8081/service/rest/v1/components?repository=petclinic" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "maven2.groupId=org.springframework.samples" -F "maven2.artifactId=petclinic" -F "maven2.version=${BUILD_ID}.0.0" -F "maven2.asset1=@${WORKSPACE}/target/petclinic.war" -F "maven2.asset1.extension=war"
          '''
      }
  }
  
  stage('Code Deploy'){
      steps {
          sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible-server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'mkdir -p //home//ansadmin//petclinic', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: ''), sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//home//ansadmin//petclinic', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*.war'), sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook /etc/ansible/deploy.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//etc//ansible//', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'deploy.yml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
      }
  }

}


post {
  always {
    echo "Always Runs the code"
  }
  success {
    echo "Only runs when its successful"
  }
  failure {
    echo "Only runs when the Job has failed"
  }
}
}
