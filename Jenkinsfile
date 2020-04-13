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
      stage('Archive') {
         steps {
            echo 'In Archive stage'
         }
      }
      stage('Deploy') {
         steps {
            echo 'In Deploy stage'
          //  mail bcc: '', body: 'build Deployed successfully', cc: '', from: '', replyTo: '', subject: 'Attn :DevOps', to: 'abinav.may24@gmail.com'         }
      
            

      }
   }
}
