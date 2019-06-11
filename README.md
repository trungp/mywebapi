# CI using Azure Dev Spaces enabled AKS

---

This tutorial shows you how to leverage [Azure Dev Spaces Plugin](https://aka.ms/azjenkinsazds) to run CI against your pull request. Below are the major steps in this tutorial.

- [Create Azure resources](#create-azure)
- [Create Azure Dev Spaces enabled AKS cluster](#create-aks)
- [Set up and deploy sample to AKS cluster](#setup-sample)
- [Prepare Jenkins server](#prepare)
- [Create job](#create-job)
- [Cretae PR to trigger CI](#create-PR)
- [Clean Up Resources](#clean-up)

## <a name="create-azure"></a>Create Azure resources

1. Install Azure CLI. If you already have the Azure CLI installed, make sure you are using version 2.0.43 or higher.
2. Create a resource group by doing: 
    
    ```az group create --name <yourResourceGroup> --location eastus```
3. Create a Azure Container Registry(ACR) to build and store Docker images: 

    ```az acr create -n <yourRegistryName> -g <yourResourceGroup> --sku <sku-name> --admin-enabled true```

## <a name="create-aks"></a>Create Azure Dev Spaces enabled AKS cluster
1. Create an AKS cluster in the resource group: 
    
    ```az aks create -g <yourResourceGroup> -n <yourAKSName> --location eastus --kubernetes-version 1.11.4 --enable-addons http_application_routing --generate-ssh-keys```

## <a name="setup-sample"></a>Set up and deploy sample to AKS Cluster
Follow the steps in [here](https://docs.microsoft.com/en-us/azure/dev-spaces/get-started-java) to set up and deploy the sample in the AKS cluster you just deployed.
1. Clone https://github.com/Azure/dev-spaces to your local machine.
2. First, set up and deploy the web front end. Make sure you are in the folder `dev-spaces/samples/java/getting-started/webfrontend`
3. Update Application.java by following the steps as illustrated in https://docs.microsoft.com/en-us/azure/dev-spaces/team-development-java#make-a-request-from-webfrontend-to-mywebapi. You edited file should look like this:
	
    ```
    package com.ms.sample.webfrontend;

    import java.io.*;
    import java.net.*;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.*;

    @SpringBootApplication
    @RestController
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @RequestMapping(value = "/greeting", produces = "text/plain")
        public String greeting(@RequestHeader(value = "azds-route-as", required = false) String azdsRouteAs) throws Exception {
            URLConnection conn = new URL("http://mywebapi/").openConnection();
            conn.setRequestProperty("azds-route-as", azdsRouteAs); // propagate dev space routing header
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream())))
            {
                return "Hello from webfrontend and " + reader.lines().reduce("\n", String::concat);
            }
        }
    }
    ```

4. Generate Docker and Helm chart assets by doing: ```azds prep --public```
5. Build and run code in AKS by doing: ```azds up```
6. Your app should be available at `http://webfrontend.XXXXXXXXXXXXXXXXXXX.eastus.aksapp.io/`
7. Next, set up and deploy mywebapi. Change directory to `dev-spaces/samples/java/getting-started/mywebapi`. Run:
	* ```azds prep –public```
	* ```azds up -d```

## <a name="prepare"></a>Prepare Jenkins server

1. If you don't already have a Jenkins master, deploy [one](https://aka.ms/jenkins-on-azure) on Azure by following steps in this [quickstart](https://docs.microsoft.com/en-us/azure/jenkins/install-jenkins-solution-template).

2. In Jenkins dashboard, install the plugins. Click 'Manage Jenkins' -> 'Manage Plugins' -> 'Available', then search and install the following plugins if not already installed: Azure Container Registry Tasks Plugin, EnvInject Plugin.
    
    Azure Dev Spaces plugin is currently in preview so download the hpi from https://aka.ms/azjenkinsazds. And then upload manually through the "Advanced" option in 'Plugin Manager'.

3. Jenkins needs an Azure service principal for autheticating and accessing Azure resources. Refer to the [Crease service principal](https://docs.microsoft.com/en-us/azure/jenkins/tutorial-jenkins-deploy-web-app-azure-app-service#create-service-principal) section in the Deploy to Azure App Service tutorial.
4. Then using the Azure service principal, add a "Microsoft Azure Service Principal" credential type in Jenkins. Refer to the [Add Service principal](https://docs.microsoft.com/en-us/azure/jenkins/tutorial-jenkins-deploy-web-app-azure-app-service#add-service-principal-to-jenkins) section in the Deploy to Azure App Service tutorial. This is the [your crendential id of service principal] mentioned in the following section
5. Set up ACR credential. Run the below az cli command to show your Azure Container Registry credentials. Add a "Username with password" credential type in Jenkins using the Docker registry username and password  
    
    ```az acr credential show -n <yourRegistryName>```

    This is [your credential id of your ACR account] in the following section.
6. Set up AKS credential. Add a "Kubernetes configuration (kubeconfig)" credential type in Jenkins (use the option "Enter directly"). You can get access credentials for your AKS by running the following az cli command: 

    ```az aks get-credentials -g <yourResourceGroup> -n <yourAKSName> -f -```
    
    This is [your kubeconfig id] in the following section.

## <a name="create-job"></a>Create job

1. Clone this [repo](https://github.com/gavinfish/mywebapi) to your own repo.
2. Add a new job in type "Pipeline".

3. Enable "Prepare an environment for the run", and add the following environment variables in "Properties Content":

    ```AZURE_CRED_ID=[your credential id of service principal]
    RES_GROUP=[your resource group of the function app]
	AZURE_CRED_ID=[your credential id of service principal]
	ACR_RES_GROUP=[your ACR resource group]
	ACR_NAME=[your ACR name]
	ACR_REGISTRY=[your ACR registry url, without http schema]
	ACR_CRED_ID=[your credential id of your ACR account]
	AKS_RES_GROUP=[your AKS resource group]
	AKS_NAME=[your AKS name]
	IMAGE_NAME=[name of Docker image you will push to ACR, without registry prefix]
	KUBE_CONFIG_ID=[your kubeconfig id]
	PARENT_DEV_SPACE=[shared dev space name]
	TEST_ENDPOINT=[your web frontend end point for testing. Should be webfrontend.XXXXXXXXXXXXXXXXXXX.eastus.aksapp.io]```
4. Choose "Pipeline script from SCM" in "Pipeline" -> "Definition".
5. In "SCM", choose "Git" and enter your repo URL
6. In Branch Specifier, enter "refs/remotes/origin/${GITHUB_PR_SOURCE_BRANCH}"
7. Fill in the SCM repo url and script path "Jenkinsfile". (Script [Example](/Jenkinsfile))
8. "Lightweight checkout" should be checked.

## <a name="create-PR"></a>Create PR to trigger CI

1. Create a PR by editing Application.java in `mywebapi/src/main/java/com/ms/sample/mywebapi/`. 

2. If you have a webhook set up (a good resource: https://dzone.com/articles/how-to-start-working-with-the-github-plugin-for-je), a job will be triggered automatically. Otherwise, you can run the job manually.  

3. Open your favorite browser and input `https://webfrontend.XXXXXXXXXXXXXXXXXXX.eastus.aksapp.io`

    Open another tab and input the PR dev space URL. Should be 
    `https://<yourdevspace>.s.webfrontend.XXXXXXXXXXXXXXXXXXX.eastus.aksapp.io`. You will find the link in the console output of the Jenkins job you just triggered.

    Notice how with the dev space prefix, your call is routed to the updated mywebapi while `https://webfrontend.XXXXXXXXXXXXXXXXXXX.eastus.aksapp.io` is still pointing to the (team's) shared version. The dev space prefix ($env.azdsprefix) is set by the Azure Dev Spaces plugin (in the 'create dev space' stage.)

    We have commented out the last stage 'cleanup' in the sample Jenkinsfile so that you can use the link to verify that space routing is working and you are running CI againts your PR build. In a reallife, you should uncomment the codes and perform this step to delete the PR dev space. 

## <a name="clean-up"></a>Clean Up Resources

Delete the Azure resources you just created by running below command:

```bash
az group delete -y --no-wait -n <yourResourceGroup>
```