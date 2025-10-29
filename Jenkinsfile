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
            az webapp deploy --resource-group "$RESOURCE_GROUP" --name "$WEBAPP_NAME" --src-path target/calculator-1.0.war --type war --async false
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
