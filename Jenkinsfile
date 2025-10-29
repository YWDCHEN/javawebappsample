@Library('') _
import groovy.json.JsonSlurper

def getFtpPublishProfile(String publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p.publishMethod == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  error('No FTP publishing profile found')
}

pipeline {
  agent any

  environment {
    AZURE_SUBSCRIPTION_ID = 'bfb0b70b-5e49-4b79-960e-766bb1e40f20'
    AZURE_TENANT_ID       = 'fb1ff8fb-edd0-483c-82b0-fa0da479f2b3'
    RESOURCE_GROUP        = 'jenkins-get-started-rg'
    WEBAPP_NAME           = 'info-sample-app-wendy1234'
  }

  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }

    stage('build') {
      steps {
        sh 'mvn -v'
        sh 'mvn clean package'
        sh 'ls -l target || true'
      }
    }

    stage('deploy') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
        )]) {
          sh '''
            set -e
            az login --service-principal -u "$AZURE_CLIENT_ID" -p "$AZURE_CLIENT_SECRET" -t "$AZURE_TENANT_ID"
            az account set -s "$AZURE_SUBSCRIPTION_ID"

            # Option A: direct deploy (preferred)
            az webapp deploy \
              --resource-group "$RESOURCE_GROUP" \
              --name "$WEBAPP_NAME" \
              --src-path target/calculator-1.0.war \
              --type war --async false

            # Option B (fallback): FTP upload using publishing profile
            # pub=$(az webapp deployment list-publishing-profiles -g "$RESOURCE_GROUP" -n "$WEBAPP_NAME")
            # echo "$pub" > pub.json
            # exit 0  # comment this line to enable the FTP fallback below
          '''
        }
      }
    }
  }

  post {
    always {
      sh 'echo "Build artifacts:" && ls -l target || true'
      sh 'az logout || true'
    }
  }
}
