def PowerShell(psCmd) {
    bat "powershell.exe -NonInteractive -ExecutionPolicy Bypass -Command \"\$ErrorActionPreference='Stop';[Console]::OutputEncoding=[System.Text.Encoding]::UTF8;$psCmd;EXIT \$global:LastExitCode\""
}
def jks_cred = 'fe75daf8-b954-41a2-8b64-195b784c059b'

pipeline {
    agent { 
        label 'windows-vagrant'
	}
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Checkout') {
            steps {
				checkout scm
            }
        }
        stage('Check-Download&Install dependencies'){
            steps{
                PowerShell(". '.\\scripts\\dependencies-sut-activedemo.ps1'") 
                echo 'Preparing Finished'
            }
        }
        stage('Clone gauge repo'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'RelativeTargetDirectory', 
                    relativeTargetDir: 'gauge'], [$class: 'CleanBeforeCheckout']], submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: "${jks_cred}", 
                    url: 'https://mtacu@bitbucket.endava.com/scm/seed/seed_poc-atf-gauge-csharp.git']]])
            }
        }
        stage('Start-SUT&Test-Run'){
            steps{
                echo 'Starting local webserver and running tests.'
                PowerShell(". '.\\scripts\\test-run.ps1'")
                echo 'Tests run success'
            }
        }
    }
    post {
        success {
            archiveArtifacts artifacts: 'gauge/html-report*.zip', onlyIfSuccessful: true
            archiveArtifacts artifacts: 'gauge/xml-report*.zip', onlyIfSuccessful: true
            publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'gauge/reports/html-report/test-report',
                reportFiles: 'index.html',
                reportName: "Test Report"
            ])
        }
    }
}