# Deploying a Jakarta EE application on Azure using Docker and Kubernetes
This demo shows how you can deploy a Jakarta EE application to Azure using Docker and Kubernetes. The following is how you run the demo.

## Setup
* You will need an Azure subscription. If you don't have one, you can get one for free for one year [here](https://azure.microsoft.com/en-us/free).

## Start Managed PostgreSQL on Azure
We will be using the fully managed PostgreSQL offering in Azure for this demo. If you have not set it up yet, please do so now. 

* Go to the [Azure portal](http://portal.azure.com).
* Select 'Create a resource'. In the search box, enter and select 'Azure Database for PostgreSQL'. Hit create. Select a single server. Hit create again.
* Create a new resource group named jakartaee-cafe-group-`<your suffix>` (the suffix could be your first name such as "reza"). Specify the Server name to be jakartaee-cafe-db-`<your suffix>` (the suffix could be your first name such as "reza"). Specify the login name to be postgres. Specify the password to be Secret123!. Create the resource. It will take a moment for the database to deploy and be ready for use.
* In the portal home, go to 'All resources'. Find and click on jakartaee-cafe-db-`<your suffix>`. Open the connection security panel. Enable access to Azure services, add the current client IP address, disable SSL connection enforcement, and then hit 'Save'.

Once you are done exploring the demo, you should delete the jakartaee-cafe-group-`<your suffix>` resource group. You can do this by going to the portal, going to resource groups, finding and clicking on jakartaee-cafe-group-`<your suffix>` and hitting delete. This is especially important if you are not using a free subscription! If you do keep these resources around (for example to begin your own prototype), you should in the least use your own passwords and make the corresponding changes in the demo code.

## Setup the Kubernetes Cluster
* You will first need to create the Kubernetes cluster. Go to the [Azure portal](http://portal.azure.com). Hit Create a resource -> Containers -> Azure Kubernetes Service. Select the resource group to be jakartaee-cafe-group-`<your suffix>`. Specify the cluster name as jakartaee-cafe-cluster-`<your suffix>` (the suffix could be your first name such as "reza"). Hit 'Review + create'. Hit 'Create'.

## Setup Kubernetes Tooling
* You need to have Docker CLI installed and you must be signed into your Docker Hub account. To create a Docker Hub account go to [https://hub.docker.com](https://hub.docker.com).
* You will now need to setup kubectl. [Here](https://kubernetes.io/docs/tasks/tools/install-kubectl/) are instructions on how to do that.
* Next you will install the Azure CLI. [Here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) are instructions on how to do that.
* You will then connect kubectl to the Kubernetes cluster you created. To do so, run the following command:

   ```
   az aks get-credentials --resource-group jakartaee-cafe-group-<your suffix> --name jakartaee-cafe-cluster-<your suffix>
   ```
  If you get an error about an already existing resource, you may need to delete the ~/.kube directory.

## Deploy the Application on Kubernetes
* Open Eclipse.
* Get the basic jakartaee-cafe application into the IDE if you have not done so already. In order to do that, go to File -> Import -> Maven -> Existing Maven Projects. Then browse to where you have this repository code in your file system and select jakartaee/jakartaee-cafe. Accept the rest of the defaults and finish.
* Do a full build of the jakartaee-cafe application via Maven by going to Right click the application -> Run As -> Maven install.
* Browse to where you have this repository code in your file system. You will now need to copy the war file to where we will build the Docker image next. You will find the war file under jakartaee/jakartaee-cafe/target. Copy the war file to kubernetes/. Open the [standalone.xml](standalone.xml) file in the kubernetes/ directory with a text editor next. Replace occurrences of `reza` with `<your suffix>`.
* Open a terminal. Navigate to where you have this repository code in your file system. Navigate to the kubernetes/ directory.
* Log in to Docker Hub using the docker login command:

   ```
   docker login
   ```
* Build the Docker image:

   ```
   docker build -t <your Docker Hub ID>/jakartaee-cafe:v1 .
   ```
   
* Test the Docker image locally:

   ```
   docker run -it --rm -p 8080:8080 <your Docker Hub ID>/jakartaee-cafe:v1
   ```
   
   The application will be available at http://localhost:8080/jakartaee-cafe/. Press Control-C to stop.
   
* Push the Docker image to Docker Hub:

   ```
   docker push <your Docker Hub ID>/jakartaee-cafe:v1
   ```

* Replace the `<your Docker Hub ID>` value with your account name in `jakartaee-cafe.yml` file.
* You can now deploy the application:

   ```
   kubectl create -f jakartaee-cafe.yml
   ```
* Get the External IP address of the Service, then the application will be accessible at `http://<External IP Address>/jakartaee-cafe`:
   ```
   kubectl get svc jakartaee-cafe --watch
   ```
  It may take a few minutes for the load balancer to be created. When the external IP changes over from *pending* to a valid IP, just hit Control-C to exit.

* Scale your application:
   ```
   kubectl scale deployment jakartaee-cafe --replicas=3
   ```
   
## Deleting the Resources
* Delete the application deployment:
   ```
   kubectl delete -f jakartaee-cafe.yml
   ```
