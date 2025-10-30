// Jenkinsfile for Azure App Service (Windows) + Java (Tomcat) + WAR deploy
node {
  // put these 3 in env so we can use them in sh blocks
  withEnv([
    'AZURE_SUBSCRIPTION_ID=bfb0b70b-5e49-4b79-960e-766bb1e40f20',
    'AZURE_TENANT_ID=fb1ff8fb-edd0-483c-82b0-fa0da479f2b3'
  ]) {

    stage('Checkout') {
      checkout scm
    }

    stage('Build (Maven)') {
      // this creates target/calculator-1.0.war
      sh 'mvn clean package'
    }

    stage('Deploy to Azure') {
      // values from your messages
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName    = 'ywdchenApp'

      // use the service principal credential you created in Jenkins
      withCredentials([
        usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
        )
      ]) {
        sh """
          # login non-interactively
          az login --service-principal \\
            --username \$AZURE_CLIENT_ID \\
            --password \$AZURE_CLIENT_SECRET \\
            --tenant   \$AZURE_TENANT_ID

          # pick subscription
          az account set --subscription \$AZURE_SUBSCRIPTION_ID

          # make sure the app is Windows Java + Tomcat
          # (for example Java 17 + Tomcat 9; adjust if your lab said 11)
          az webapp config set \\
            --resource-group ${resourceGroup} \\
            --name ${webAppName} \\
            --java-version 17 \\
            --java-container Tomcat \\
            --java-container-version 9.0

          # actually deploy the WAR (this is the key change!)
          az webapp deploy \\
            --resource-group ${resourceGroup} \\
            --name ${webAppName} \\
            --src-path target/calculator-1.0.war \\
            --type war \\
            --target-path ROOT.war

          # restart to pick everything up
          az webapp restart --resource-group ${resourceGroup} --name ${webAppName}

          # logout to be clean
          az logout
        """
      }
    }
  }
}
