node {
    stage('init') {
        checkout scm
    }

    stage('build') {
        acrQuickTask azureCredentialsId: '438bf5b7-50d9-413d-876f-1f2ad7b1c650', 
            imageNames: [[image: 'mywebapi:latest']], 
            registryName: 'jiesheacr', 
            resourceGroupName: 'jiesheacr', 
            dockerfile: 'Dockerfile.develop'
    }

    stage('deploy') {
        devSpaces aksName: 'jiesheaks', 
            azureCredentialsId: '438bf5b7-50d9-413d-876f-1f2ad7b1c650', 
            endpointVariable: '', 
            helmChartLocation: 'charts/mywebapi', 
            imageRepository: 'jiesheacr.azurecr.io/mywebapi', 
            imageTag: 'latest', 
            kubeconfigId: 'adskubeconfig', 
            resourceGroupName: 'jiesheaks', 
            secretName: 'registrysecret', 
            secretNamespace: 'jieshe', 
            sharedSpaceName: 'default', 
            spaceName: 'jieshe',
            dockerCredentials: [[credentialsId: 'acr']]
    }
}