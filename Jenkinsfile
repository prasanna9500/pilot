pipeline {
   agent any

    // creating environment variables to make jenkinsfile declarative
    environment {
        //ARTIFACTORY_CREDS = credentials('svcartifactorycicd')
               //def Application_Name = ${package}
               def Artifactory_Bucket = "PFJ-NuGet-Packages"
               def Artifactory_File = ""
               def versionExists = 'false'
    }

    // display all environment variables in jenkins console output
    stages {
        stage('Echo Environment Variables') {
            steps {
                // output env variables
                
                echo "GITHUB_BRANCH: ${branch}"
                
            }
        }
        stage('Checkout Git Branch') {
            steps {
                               checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: '${repo}']]])
                               }
        }
        // SQ for code quality, MSBuild/dotnet debug build to verify solution is ready for deployment.
        stage ('DotNet Core Build (Linux)') {   
               when {
                       expression { technology == 'dotnetcore' && agent == 'linux'}
               }              
               agent { label 'linux' }
               steps {
                       sh 'dotnet clean'
                       sh 'dotnet publish -c Release -r win-x64'
               }
        }
        stage ('DotNet Core Build (Windows)') {   
               when {
                       expression { technology == 'dotnetcore' && agent == 'linux'}
               }              
               agent { label 'linux' }
               steps {
                       bat 'dotnet clean'
                       bat 'dotnet publish -c Release -r win-x64'
               }
        }
        stage ('Java Build (Windows)') {   
               when {
                       expression { technology == 'java' && agent == 'windows'}
               }              
               agent { label 'windows' }
               steps {
                       bat 'mvn clean package'
                       
               }
        }
        
        // Check if the build version has already been created. If so, fail the job. We don't want versions over-written
       
        stage('Validate Build Version') {            
               agent { label 'linux' }
               steps {
                               
                       sh "echo 'Validate Build Version'"
                    
               }
       }
        // Release artifacts are pushed to Artifactory. 
        stage ('Publish to Artifactory') {
               when {
                expression { versionExists == 'false' }
               }
               agent { label 'linux' }
               steps {
               
                               sh "echo 'Publish'"
                    
               }
        }     
                     
    }
    post {
        success {
            // send job complete email on success
            emailext (
                subject: "DEPLOYMENT COMPLETE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """SUCCESSFUL: Jenkins Job -- ${env.JOB_NAME} [${env.BUILD_NUMBER}]\n\nCheck the console output at https://18.216.131.49:9999/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/console""",
                recipientProviders: [requestor()]
            )
        }
        failure {
            // send job failure email on failure
            emailext (
                subject: "DEPLOYMENT FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """BUILD MARKED AS FAILURE: Jenkins Job -- ${env.JOB_NAME} [${env.BUILD_NUMBER}]\n\nCheck the console output at https://18.216.131.49:9999/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/console""",
                recipientProviders: [requestor()]
           )
        }
            
        always {

           cleanWs()

         }
    }
}


