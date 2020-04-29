
            // Artifactory server id configured in the jenkins along with credentials
            artifactoryServer = Artifactory.server 'Artifactory-oss-5.10.3'
       def artifactoryPublishInfo

    

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

