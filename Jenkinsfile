import groovy.json.JsonSlurper

def getFtpPublishProfile(String publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  return null
}

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=bfb0b70b-5e49-4b79-960e-766bb1e40f20',
    'AZURE_TENANT_ID=fb1ff8fb-edd0-483c-82b0-fa0da479f2b3'
  ]) {

    stage('init') {
      checkout scm
    }

    stage('build') {
      // Skip tests to speed up first build; remove -DskipTests if you want tests
      sh 'mvn clean package -DskipTests'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName   = 'info-sample-app-wendy1234'

      // Login to Azure using stored Jenkins credentials
      withCredentials([usernamePassword(
        credentialsId: 'AzureServicePrincipal',
        usernameVariable: 'AZURE_CLIENT_ID',
        passwordVariable: 'AZURE_CLIENT_SECRET'
      )]) {
        sh '''#!/usr/bin/env bash
          set -euo pipefail
          az login --service-principal -u "$AZURE_CLIENT_ID" -p "$AZURE_CLIENT_SECRET" -t "$AZURE_TENANT_ID"
          az account set -s "$AZURE_SUBSCRIPTION_ID"
        '''
      }

      // (Optional) Get publishing profiles as JSON if you want to use FTP later
      def pubProfilesJson = sh(
        script: "az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName} -o json",
        returnStdout: true
      ).trim()
      // def ftpProfile = getFtpPublishProfile(pubProfilesJson) // unused unless you switch to FTP

      // Deploy the WAR via Azure CLI (recommended)
      sh """#!/usr/bin/env bash
        set -euo pipefail
        az webapp deploy \
          --resource-group ${resourceGroup} \
          --name ${webAppName} \
          --type war \
          --src-path target/calculator-1.0.war
      """

      sh 'az logout || true'
    }
  }
}
