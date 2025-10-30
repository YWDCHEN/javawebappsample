// Jenkinsfile – build Java with Maven and deploy WAR to Azure App Service (Tomcat)

pipeline {
  agent any

  environment {
    // your subscription + tenant
    AZURE_SUBSCRIPTION_ID = 'bfb0b70b-5e49-4b79-960e-766bb1e40f20'
    AZURE_TENANT_ID       = 'fb1ff8fb-edd0-483c-82b0-fa0da479f2b3'

    // target app
    AZ_RESOURCE_GROUP = 'jenkins-get-started-rg'
    AZ_WEBAPP_NAME    = 'ywdchenApp'           // make sure this matches your app in Azure
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (Maven)') {
      steps {
        sh 'mvn -B clean package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        }
      }
    }

    stage('Azure login') {
      steps {
        // this assumes you created a Jenkins credential with ID 'AzureServicePrincipal'
        // kind: "Username with password"
        // username = appId (client id), password = client secret
        withCredentials([usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
        )]) {
          sh '''
            az login --service-principal \
              -u "$AZURE_CLIENT_ID" \
              -p "$AZURE_CLIENT_SECRET" \
              -t "$AZURE_TENANT_ID"
            az account set -s "$AZURE_SUBSCRIPTION_ID"
          '''
        }
      }
    }

    stage('Deploy to Azure (WAR)') {
      steps {
        // pick the war we just built
        script {
          // change this if your artifact name is different
          def warFile = 'target/calculator-1.0.war'
          sh """
            az webapp deploy \
              --resource-group "$AZ_RESOURCE_GROUP" \
              --name "$AZ_WEBAPP_NAME" \
              --src-path "$warFile" \
              --type war
          """
        }
      }
    }
  }

  post {
    always {
      sh 'az logout || true'
    }
  }
}
