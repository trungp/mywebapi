node {
    // Variable to record user's input
    def userInput

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
            spaceName: env.GITHUB_PR_SOURCE_BRANCH
    }

    stage('deploy') {
        kubernetesDeploy deployTypeClass: [configs: 'kubeconfigs/**'],
            dockerCredentials: [[credentialsId: env.ACR_CRED_ID, url: "http://$env.ACR_REGISTRY"]],
            kubeconfigId: env.KUBE_CONFIG_ID,
            secretNamespace: env.GITHUB_PR_SOURCE_BRANCH
    }

    stage('smoketest') {
        // CI testing against http://$env.azdsprefix.$env.TEST_ENDPOINT" 
        SLEEP_TIME = 30
        SITE_UP = false
        for (int i = 0; i < 10; i++) {
            sh "sleep ${SLEEP_TIME}"
            code = sh(returnStdout: true, script: "curl -sL -w '%{http_code}' 'http://$env.azdsprefix.$env.TEST_ENDPOINT/greeting' -o /dev/null").trim()
            if (code == 200) {
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
                id: 'Proceed1', message: 'Do you want to continue?', parameters: [
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

    } else {
        // Send a notification
    } 

    stage('cleanup') {
        // devSpacesCleanup aksName: env.AKS_NAME, 
        //     azureCredentialsId: env.AZURE_CRED_ID, 
        //     devSpaceName: env.GITHUB_PR_SOURCE_BRANCH, 
        //     kubeConfigId: env.KUBE_CONFIG_ID, 
        //     resourceGroupName: env.AKS_RES_GROUP
    }
}
