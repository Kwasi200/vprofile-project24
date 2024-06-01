/*def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
] */
pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8" 
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot-24'
		NEXUS_USER = 'admin'
		NEXUS_PASS = '1010'
		RELEASE_REPO = 'vprofile-release24'
		CENTRAL_REPO = 'vpro-maven-central-24'
		NEXUSIP = '10.0.12.178'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'vpro-maven-group-24'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        NEXUSPASS = credentials('nexuslogin')
        CURRENT_TIME = ''
        ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
        AWS_S3_BUCKET = 'vprocicdbean-24'
        AWS_EB_APP_NAME = 'vproapp'
        AWS_EB_ENVIRONMENT = 'Vproapp-env'
        AWS_EB_APP_VERSION = "${BUILD_ID}"

    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Set Time") {
            steps {
                script {
                    // Get the current date and time
                    def currentTime = new Date()
                    def formattedTime = currentTime.format("yyyy-MM-dd_HH-mm-ss")

                    // Set the environment variable
                    env.CURRENT_TIME = formattedTime

                    // Print the current time
                    echo "Current Time: ${env.CURRENT_TIME}"
                }
            }
        }
        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.CURRENT_TIME}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}",
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }

        stage('Deploy to Stage Bean'){
            steps {
                withAWS(credentials: 'awsbeancreds', region: 'eu-west-2') {
                sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION'
                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
 
            }
        }
        }

        
    }
   
}