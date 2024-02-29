pipeline {
  agent any
  stages {
    stage('BUILD') {
      post {
        success {
          echo 'Now Archiving...'
          archiveArtifacts '**/target/*.war'
        }

      }
      steps {
        sh 'mvn clean install -DskipTests'
      }
    }

    stage('UNIT TEST') {
      steps {
        sh 'mvn test'
      }
    }

    stage('INTEGRATION TEST') {
      steps {
        sh 'mvn verify -DskipUnitTests'
      }
    }

    stage('CODE ANALYSIS WITH CHECKSTYLE') {
      post {
        success {
          echo 'Generated Analysis Result'
        }

      }
      steps {
        sh 'mvn checkstyle:checkstyle'
      }
    }

    stage('CODE ANALYSIS with SONARQUBE') {
      environment {
        scannerHome = 'SonarScanner'
      }
      post {
        always {
          timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate true
          }

        }

      }
      steps {
        withSonarQubeEnv('sonarserver') {
          sh '${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile                         -Dsonar.projectName=vprofile-repo                         -Dsonar.projectVersion=1.0                         -Dsonar.sources=src/                         -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/                         -Dsonar.junit.reportsPath=target/surefire-reports/                         -Dsonar.jacoco.reportsPath=target/jacoco.exec                         -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'
        }

      }
    }

    stage('UPLOAD ARTIFACT') {
      steps {
        nexusArtifactUploader(nexusVersion: 'nexus3', protocol: 'http', nexusUrl: "${NEXUSIP}:${NEXUSPORT}", groupId: 'QA', version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}", repository: "${RELEASE_REPO}", credentialsId: "${NEXUS_LOGIN}", artifacts: [
                                                          [artifactId: 'vproapp' ,
                                                          classifier: '',
                                                          file: 'target/vprofile-v2.war',
                                                          type: 'war']
                                                      ])
        }
      }

    }
    tools {
      maven 'maven'
    }
    environment {
      SNAP_REPO = 'Snapshot'
      RELEASE_REPO = 'Release'
      CENTRAL_REPO = 'Proxy'
      NEXUSIP = 'nexus'
      NEXUSPORT = '8081'
      NEXUS_GRP_REPO = 'group'
      NEXUS_LOGIN = 'nexuslogin'
    }
  }