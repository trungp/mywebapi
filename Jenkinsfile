node {
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

    stage('deploy') {
        devSpacesCreate aksName: env.AKS_NAME, 
            azureCredentialsId: env.AZURE_CRED_ID, 
            kubeconfigId: env.KUBE_CONFIG_ID, 
            resourceGroupName: env.AKS_RES_GROUP, 
            sharedSpaceName: env.PARENT_DEV_SPACE, 
            spaceName: env.DEV_SPACE


        // kubernetesDeploy deployTypeClass: [helmChartLocation: 'charts/mywebapi', helmNamespace: 'scott', helmReleaseName: 'releasename'], 
        //     kubeconfigId: 'adskubeconfig', 
        //     secretName: ''
        kubernetesDeploy deployTypeClass: [configs: 'kubeconfigs/**'],
            dockerCredentials: [[credentialsId: env.ACR_CRED_ID, url: "http://$env.ACR_REGISTRY"]],
            kubeconfigId: env.KUBE_CONFIG_ID,
            secretNamespace: env.DEV_SPACE
    }

    stage('smoketest') {
        // CI testing against http://$env.azdsprefix.$env.TEST_ENDPOINT" 
    }
      
    stage('confirm merge') {
        // This is just an example for how an email can be triggered… can be anything and up to customers to define.
        // mail (to: 'to@example.com',
        // subject: "Job ${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input",
        // body: "Please go to ${env.BUILD_URL}.")
        // input 'Ready to go?'
    }

    stage('deploy') {
        // Apply the deployment to shared namespace in AKS using acsDeploy, Helm or kubectl   
    }
      
    stage('Verify') {
        // verify the staging environment is working properly
    }


    stage('test') {
        sh "echo http://$env.azdsprefix.$env.TEST_ENDPOINT"
    }

    stage('cleanup') {
        // devSpacesCleanup aksName: 'demoaks', 
        //     azureCredentialsId: 'jenkins-sp', 
        //     devSpaceName: 'scott', 
        //     kubeConfigId: 'adskubeconfig', 
        //     resourceGroupName: 'demo-aks'
    }
}