node('master') {
    // Variable to record user's input
    def userInput
    def devSpaceNamespace = "$env.GITHUB_PR_SOURCE_BRANCH-helm"
    def releaseName = "mywebapi${devSpaceNamespace}"

    stage('init') {
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
        kubernetesDeploy deployType: 'helm', 
            deployTypeClass: [helmCommandClass: [helmChartLocation: 'charts/mywebapi', helmChartType: 'uri', helmNamespace: "${devSpaceNamespace}", helmReleaseName: "${releaseName}", 
                setValues: "image.repository=$env.ACR_REGISTRY/$env.IMAGE_NAME,image.tag=$env.BUILD_NUMBER,ingress.hosts={localhost},imagePullSecrets[0].name=helm-secret-16"], 
            helmCommandType: 'install', 
            helmWait: true, 
            tillerNamespace: 'azds'], 
            kubeconfigId: 'adskubeconfig'
    }

    stage('smoketest') {
        // CI testing against http://$env.azdsprefix.$env.TEST_ENDPOINT" 
        SLEEP_TIME = 30
        SITE_UP = false
        for (int i = 0; i < 10; i++) {
            sh "sleep ${SLEEP_TIME}"
            code = "0"
            try {
                code = sh returnStdout: true, script: "curl -sL -w '%{http_code}' 'http://$env.azdsprefix.$env.TEST_ENDPOINT/greeting' -o /dev/null"
            } catch (Exception e){
                // ignore
            }
            if (code == "200") {
                sh "curl http://$env.azdsprefix.$env.TEST_ENDPOINT/greeting"
                SITE_UP = true
                break
            }
        }
        if(!SITE_UP) {
            echo "The site has not been up after five minutes"
        }
    }
      
    stage('confirm merge') {
        // This is just an example for how an email can be triggered… can be anything and up to customers to define.
        // mail (to: 'to@example.com',
        // subject: "Job ${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input",
        // body: "Please go to ${env.BUILD_URL}.")
        // input 'Ready to go?'

        // Wait for users to decide whether continue the process
        try {
            userInput = input(
                id: 'Proceed1', message: 'Do you want to apply changes in shared namespace and delete private dev space?', parameters: [
                [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you want to continue the process']
                ])
        } catch(err) { // input false
            echo "Aborted"
        }
    }

    if (userInput == true) {
        stage('deploy') {
            // Apply the deployment to shared namespace in AKS using acsDeploy, Helm or kubectl   
        }
        
        stage('Verify') {
            // verify the staging environment is working properly
        }

        stage('cleanup') {
            devSpacesCleanup aksName: env.AKS_NAME, 
                azureCredentialsId: env.AZURE_CRED_ID, 
                devSpaceName: devSpaceNamespace, 
                kubeConfigId: env.KUBE_CONFIG_ID, 
                resourceGroupName: env.AKS_RES_GROUP,
                helmReleaseName: releaseName
        }

    } else {
        // Send a notification
    } 

}
