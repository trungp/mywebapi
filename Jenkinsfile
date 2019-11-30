node('master') {
    // Variable to record user's input
    def userInput
    def devSpaceNamespace = "$env.namespace"
    def releaseName = "mywebapi${devSpaceNamespace}"

    stage('init') {
        cleanWs()
        checkout scm
    }

    stage('build') {
        acrQuickTask azureCredentialsId: env.AZURE_CRED_ID, 
            imageNames: [[image: "$env.ACR_REGISTRY/$env.IMAGE_NAME:$env.BUILD_NUMBER"]], 
            registryName: env.ACR_NAME, 
            resourceGroupName: env.ACR_RES_GROUP, 
            dockerfile: 'Dockerfile.develop'
    }

    stage('create dev space') {
        devSpacesCreate aksName: env.AKS_NAME, 
            azureCredentialsId: env.AZURE_CRED_ID, 
            kubeconfigId: env.KUBE_CONFIG_ID, 
            resourceGroupName: env.AKS_RES_GROUP, 
            sharedSpaceName: env.PARENT_DEV_SPACE, 
            spaceName: devSpaceNamespace
    }

    stage('deploy') {
        def registrySecretName = "helm-secret-$env.BUILD_NUMBER"
        sh "/usr/local/bin/kubectl create secret docker-registry ${registrySecretName} --docker-server=$env.ACR_REGISTRY --docker-username=$env.ACR_NAME --docker-password=$env.ACR_SECRET --namespace=${devSpaceNamespace}"
        sh "/usr/local/bin/helm init --tiller-namespace dev"
        sh "/usr/local/bin/helm upgrade ${releaseName} ./charts/mywebapi --install --force --namespace ${devSpaceNamespace} --set image.repository=$env.ACR_REGISTRY/$env.IMAGE_NAME --set image.tag=$env.BUILD_NUMBER --set ingress.hosts={localhost}  --set imagePullSecrets[0].name=${registrySecretName} --tiller-namespace dev"
    }

    // stage('smoketest') {
    //     // CI testing against http://$env.azdsprefix.$env.TEST_ENDPOINT" 
    //     SLEEP_TIME = 30
    //     SITE_UP = false
    //     for (int i = 0; i < 10; i++) {
    //         sh "sleep ${SLEEP_TIME}"
    //         code = "0"
    //         try {
    //             code = sh returnStdout: true, script: "curl -sL -w '%{http_code}' 'http://$env.azdsprefix.$env.TEST_ENDPOINT/greeting' -o /dev/null"
    //         } catch (Exception e){
    //             // ignore
    //         }
    //         if (code == "200") {
    //             sh "curl http://$env.azdsprefix.$env.TEST_ENDPOINT/greeting"
    //             SITE_UP = true
    //             break
    //         }
    //     }
    //     if(!SITE_UP) {
    //         echo "The site has not been up after five minutes"
    //     }
    // }
      
    // stage('confirm merge') {
    //     // This is just an example for how an email can be triggered… can be anything and up to customers to define.
    //     // mail (to: 'to@example.com',
    //     // subject: "Job ${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input",
    //     // body: "Please go to ${env.BUILD_URL}.")
    //     // input 'Ready to go?'

    //     // Wait for users to decide whether continue the process
    //     try {
    //         userInput = input(
    //             id: 'Proceed1', message: 'Do you want to apply changes in shared namespace and delete private dev space?', parameters: [
    //             [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you want to continue the process']
    //             ])
    //     } catch(err) { // input false
    //         echo "Aborted"
    //     }
    // }

    // if (userInput == true) {
    //     stage('deploy') {
    //         // Apply the deployment to shared namespace in AKS using acsDeploy, Helm or kubectl   
    //     }
        
    //     stage('Verify') {
    //         // verify the staging environment is working properly
    //     }

    //     stage('cleanup') {
    //         devSpacesCleanup aksName: env.AKS_NAME, 
    //             azureCredentialsId: env.AZURE_CRED_ID, 
    //             devSpaceName: devSpaceNamespace, 
    //             kubeConfigId: env.KUBE_CONFIG_ID, 
    //             resourceGroupName: env.AKS_RES_GROUP,
    //             helmReleaseName: releaseName
    //     }

    // } else {
    //     // Send a notification
    // } 

}
