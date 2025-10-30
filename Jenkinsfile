node {
  // set your Azure IDs in env so we can use them in shell
  withEnv([
    'AZURE_SUBSCRIPTION_ID=bfb0b70b-5e49-4b79-960e-766bb1e40f20',
    'AZURE_TENANT_ID=fb1ff8fb-edd0-483c-82b0-fa0da479f2b3'
  ]) {

    stage('Checkout') {
      // pulls the repo Jenkins is pointed at
      checkout scm
    }

    stage('Build with Maven') {
      // build the Java app and produce target/calculator-1.0.war
      sh 'mvn -B -DskipTests clean package'
    }

    stage('Deploy to Azure') {
      // your Azure resource names
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName    = 'ywdchenApp'

      // pull the service principal creds from Jenkins
      withCredentials([
        usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
        )
      ]) {
        // IMPORTANT: we use az webapp deploy — no FTP, no publish profile
        sh """
          set -e

          # login to Azure using the service principal from Jenkins
          az login --service-principal \\
            --username $AZURE_CLIENT_ID \\
            --password $AZURE_CLIENT_SECRET \\
            --tenant $AZURE_TENANT_ID

          # pick the subscription you told me
          az account set --subscription $AZURE_SUBSCRIPTION_ID

          # deploy the WAR that Maven just built
          az webapp deploy \\
            --resource-group ${resourceGroup} \\
            --name ${webAppName} \\
            --src-path target/calculator-1.0.war \\
            --type war

          # logout to be tidy
          az logout
        """
      }
    }
  }
}
