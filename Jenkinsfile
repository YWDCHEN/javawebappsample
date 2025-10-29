import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=bfb0b70b-5e49-4b79-960e-766bb1e40f20',
        'AZURE_TENANT_ID=fb1ff8fb-edd0-483c-82b0-fa0da479f2b3']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
   stage('deploy') {
  withEnv([
    'RESOURCE_GROUP=jenkins-get-started-rg',
    'WEBAPP_NAME=info-sample-app-wendy1234'
  ]) {
    withCredentials([usernamePassword(
      credentialsId: 'AzureServicePrincipal',
      passwordVariable: 'AZURE_CLIENT_SECRET',
      usernameVariable: 'AZURE_CLIENT_ID'
    )]) {
      sh '''
        set -e
        az login --service-principal -u "$AZURE_CLIENT_ID" -p "$AZURE_CLIENT_SECRET" -t "$AZURE_TENANT_ID"
        az account set -s "$AZURE_SUBSCRIPTION_ID"
        az webapp deploy --resource-group "$RESOURCE_GROUP" --name "$WEBAPP_NAME" --src-path target/calculator-1.0.war --type war --async false
        az logout
      '''
    }
  }
}

