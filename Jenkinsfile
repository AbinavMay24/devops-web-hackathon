def projectProperties = [
        buildDiscarder(logRotator(artifactDaysToKeepStr: '20', artifactNumToKeepStr: '20', daysToKeepStr: '20', numToKeepStr: '20')),
        [$class: 'GithubProjectProperty', projectUrlStr: 'https://github.com/veersudhir83/devops-web-maven.git/']
        //,pipelineTriggers([pollSCM('H/10 * * * *')])
]

properties(projectProperties)

try {
    node {
        def mvnHome
        def mvnAnalysisTargets = '-P metrics pmd:pmd test javadoc:javadoc '
        def antHome
        def artifactoryPublishInfo
        def artifactoryDownloadInfo
        def artifactoryServer
        def isArchivalEnabled = true //params.IS_ARCHIVAL_ENABLED // Enable if you want to archive files and configs to artifactory
        def isSonarAnalysisEnabled = true //params.IS_ANALYSIS_ENABLED // Enable if you want to analyze code with sonarqube
        def isDeploymentEnabled = true //params.IS_DEPLOYMENT_ENABLED // Enable if you want to deploy code on app server
        def isSeleniumTestingEnabled = true //params.IS_SELENIUM_TESTING_ENABLED // Enable if you want to generate reports
        def isReportsEnabled = true //params.IS_REPORTS_ENABLED // Enable if you want to generate reports

        def appName = 'devops-web-maven'// application name currently in progress
        def appEnv  // application environment currently in progress
        def artifactName = appName // name of the war/jar/ear in artifactory
        def artifactExtension = "jar" // extension of the war/jar/ear - for both target directory and artifactory
        def artifactoryRepoName = 'DevOps' // repo name in artifactory
        def artifactoryAppName = appName // application name as per artifactory

        def buildNumber = env.BUILD_NUMBER
        def workspaceRoot = env.WORKSPACE
        def artifactoryTempFolder = 'downloadsFromArtifactory' // name of the local temp folder where file(s) from artifactory get downloaded
        def sonarHome
        def SONAR_HOST_URL = 'http://localhost:9000'

        // Logic for Slack Notification Service
        def slackBaseUrl = 'https://defaultgrouptalk.slack.com/services/hooks/jenkins-ci/'
        def slackChannel = '#general'
        def slackTeamDomain = 'defaultgrouptalk'
        def slackMessagePrefix = "Job ${env.JOB_NAME}:${env.BUILD_NUMBER}"
        def slackTokenCredentialId = 'ecd292a7-bf0e-45c9-b599-aeb317ce2170' // replace with right one from jenkins credentials details

        // color can be good, warning, danger or anything
        slackSend baseUrl: "${slackBaseUrl}", channel: "${slackChannel}", color: "good", message: "${slackMessagePrefix} -> Build Started", teamDomain: "${slackTeamDomain}", tokenCredentialId: "${slackTokenCredentialId}"


        if (isArchivalEnabled) {
            // Artifactory server id configured in the jenkins along with credentials
            artifactoryServer = Artifactory.server 'Artifactory'
        }

        // to download appConfig.json files from artifactory
        def downloadAppConfigUnix = """{
            "files": [
                {
                    "pattern": "generic-local/Applications/${artifactoryRepoName}/${artifactoryAppName}/${appEnv}/appConfig.json",
                    "target": "${workspaceRoot}/${artifactoryTempFolder}/"
                }
            ]
        }"""

        def downloadAppConfigWindows = """{
            "files": [
                {
                    "pattern": "generic-local/Applications/${artifactoryRepoName}/${artifactoryAppName}/${appEnv}/appConfig.json",
                    "target": "${workspaceRoot}/${artifactoryTempFolder}/"
                }
            ]
        }"""

        def uploadAppConfigUnix = """{
            "files": [
                {
                    "pattern": "${workspaceRoot}/${artifactoryTempFolder}/${artifactoryAppName}/${appEnv}/appConfig.json",
                    "target": "generic-local/Applications/${artifactoryRepoName}/${artifactoryAppName}/${appEnv}/"
                }
            ]
        }"""

        def uploadAppConfigWindows = """{
            "files": [
                {
                    "pattern": "${workspaceRoot}\\${artifactoryTempFolder}\\${artifactoryAppName}\\${appEnv}\\appConfig.json",
                    "target": "generic-local/Applications/${artifactoryRepoName}/${artifactoryAppName}/${appEnv}/"
                }
            ]
        }"""

        def uploadMavenArtifactUnix = """{
            "files": [
                {
                    "pattern": "${workspaceRoot}/${appName}/target/${artifactName}.${artifactExtension}",
                    "target": "generic-local/Applications/${artifactoryRepoName}/${artifactoryAppName}/app/${buildNumber}/"
                }
            ]
        }"""

        def uploadMavenArtifactWindows = """{
            "files": [
                {
                    "pattern": "${workspaceRoot}\\${appName}\\target\\${artifactName}.${artifactExtension}",
                    "target": "generic-local/Applications/${artifactoryRepoName}/${artifactoryAppName}/app/${buildNumber}/"
                }
            ]
        }"""


pipeline {
   agent any
    tools {
      maven 'mvn3.5.4'
    }
    
    triggers {
      cron 'H/30 H * * * '
    }


   stages {
      stage('Checkout') {
         steps {
            //cleanWs() 
            echo 'In Checkout Stage'
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/veersudhir83/devops-web-hackathon.git']]])
         }
      }
      stage('Build') {
         steps {
            echo 'In Build stage'
            sh label: '', script: 'mvn --version && mvn clean package'
         }
      }
      stage('Analyze') {
         steps {
            echo 'In Analyze Stage'
            sh label: '', script: 'mvn --version && mvn -P metrics pmd:pmd test sonar:sonar'
         }
      }
         stage('Publish') {
            try {
                if (isArchivalEnabled) {
                    echo 'Publish Artifacts & appConfig.json in progress'
                    if (isUnix()) {
                        dir('devops-web-hackathon/') {
                            if (fileExists('target/devops-web-hackathon.jar')) {
                                // upload artifactory and also publish build info
                                artifactoryPublishInfo = artifactoryServer.upload(uploadMavenArtifactUnix)
                                artifactoryPublishInfo.retention maxBuilds: 5
                                // and publish build info to artifactory
                                artifactoryServer.publishBuildInfo(artifactoryPublishInfo)
                            } else {
                                error 'Publish: Failed during file upload/publish to artifactory'
                            }
                        }
                        // TODO: Work on this
                        /*causing Error: Error occurred for request CONNECT localhost:8081 HTTP/1.1:
                        sun.security.validator.ValidatorException: PKIX path building failed:
                        sun.security.provider.certpath.SunCertPathBuilderException:
                        unable to find valid certification path to requested target.*/

                        /*
                        artifactoryServer.download(downloadAppConfigUnix)
                        dir('downloadsFromArtifactory/') {
                            sh '''
                                curl -uadmin:APTvW3dVn6kUTbS -O "http://localhost:8081/artifactory/generic-local/Applications/DevOps/devops-web-maven/DEV/appConfig.json"
                                FILE=appConfig.json
                                TEMP=temp.json
                                if [ -f $FILE ]
                                then
                                echo "File $FILE exists."
                                mv $FILE $TEMP
                                command_publish="jq '.component[0].Build_Number = ${BUILD_NUMBER}' $TEMP > $FILE"
                                eval $command_publish
                                fi
                            '''
                        }
                        */
                    } else {
                        dir('devops-web-hackathon\\') {
                            if (fileExists('target\\devops-web-hackathon.jar')) {
                                // upload artifactory and also publish build info
                                artifactoryPublishInfo = artifactoryServer.upload(uploadMavenArtifactWindows)
                                artifactoryPublishInfo.retention maxBuilds: 5
                                // and publish build info to artifactory
                                artifactoryServer.publishBuildInfo(artifactoryPublishInfo)
                            } else {
                                error 'Publish: Failed during file upload/publish to artifactory'
                            }
                        }
                    }
                }
                slackSend color: "good", message: "${slackMessagePrefix} -> Archival Complete"
            } catch (exc) {
                slackSend color: "danger", message: "${slackMessagePrefix} -> Archival Failed"
                error "Failure in Publish stage: ${exc}"
            }
        }
      stage('Deploy') {
         steps {
            echo 'In Deploy stage'
             }
      
            

      }
   }
}
