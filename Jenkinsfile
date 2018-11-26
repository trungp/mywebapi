node {
    stage('init') {
        checkout scm
    }

    stage('build') {
        acrQuickTask azureCredentialsId: env.AZURE_CRED_ID, 
            imageNames: [[image: '$env.ACR_REGISTRY/$env.IMAGE_NAME:$env.BUILD_NUMBER']], 
            registryName: env.ACR_NAME, 
            resourceGroupName: env.ACR_RES_GROUP, 
            dockerfile: 'Dockerfile.develop'
    }

    stage('deploy') {
        devSpaces aksName: env.AKS_NAME, 
            azureCredentialsId: env.AZURE_CRED_ID, 
            endpointVariable: '', 
            helmChartLocation: 'charts/$env.IMAGE_NAME', 
            imageRepository: '$env.ACR_REGISTRY/$env.IMAGE_NAME', 
            imageTag: env.BUILD_NUMBER, 
            kubeconfigId: env.KUBE_CONFIG_ID, 
            resourceGroupName: env.AKS_RES_GROUP, 
            secretName: env.SECRET_NAME, 
            secretNamespace: env.SECRET_NAMESPACE, 
            sharedSpaceName: env.PARENT_DEV_SPACE, 
            spaceName: env.DEV_SPACE,
            dockerCredentials: [[credentialsId: env.ACR_CRED_ID, url: 'https://jiesheacr.azurecr.io']]
    }
}