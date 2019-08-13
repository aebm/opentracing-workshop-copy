Welcome to Lab 02 - Kubernetes Setup
===

In this lab we'll be setting up and configuring Google Kubernetes Engine (GKE). We'll also gather tokens and keys needed to configure and integrate with Gitlab so we can build and deploy our sample microservice application

## Tasks

- [ ] 1 :: [Setup Kubernetes Cluster](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#setup-kubernetes-cluster)
  - [ ] 1.1 :: [Enable Billing](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#11-enable-billing)
  - [ ] 1.2 :: [Create Cluster](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#12-create-cluster)
  - [ ] 1.3 :: [Connect to Cloud Shell](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#13-connect-to-cloud-shell)
  - [ ] 1.4 :: [Generate External kubeconfig](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#14-generate-external-kubeconfig)
- [ ] 2 :: [Configure Kubernetes Cluster](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#configure-kubernetes-cluster)
  - [ ] 2.1 :: [Setup Cluster](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#21-setup-cluster)
  - [ ] 2.2 :: [Export Gitlab Credentials](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#22-export-gitlab-credentials)

Setup Kubernetes Cluster
---

### 1.1 Enable Billing

> Click on the Kubernetes Engine link, under the Compute section, on the left nav bar. We'll be prompted to enable billing, this is required in order to utilize the $300 credit we just recieved from Google.

![Enable Billing](/lab-02/images/img01.png)

### 1.2 Create Cluster 

> This will take a few minutes, once the API is ready, we can click the `Create Cluster` button, leave all values default except the following and fill with these values:
> 
> * Number of Nodes: 4
> * Machine type: 2 vCPUs (7.5GB Memory)

![Create Cluster](/lab-02/images/img02.png)

### 1.3 Connect to Cloud Shell 

> Once our cluster is finished being created, we will need to click the `Connect` button, then click `Run in Cloud Shell`, confirm that action by clicking `Start Cloud Shell`
> 
> It will take a few moments for the shell to be provisioned, once finished there will be a command line prompt waiting for us with `gcloud container clusters get-credentials ...` pre-populated for us. Press the Enter key on the keyboard to execute this command.

![Connect to Cluster](/lab-02/images/img03.gif)

![Get Credentials](/lab-02/images/img04.png)

### 1.4 Generate External kubeconfig

> We are performing this step because the docker container we'll be using to configure the cluster is not compatible with the authentication provider which comes prebaked into Cloud Shell. In order to access the Kubernetes API we generate a valid kubeconfig file which can be volume mounted into the docker container which will configure our cluster for deployment using Gitlab.

> Once inside the cloud shell, we must execute this shell script to generate the kubeconfig file, run: `curl -Ls https://git.io/fj9JB | bash -s` in the terminal window. Once the command has executed we can inspect the new kubeconfig file by executing `cat ~/build/kubeconfig`

![Create kubeconfig](/lab-02/images/img05.gif)

Configure Kubernetes Cluster
---

### 2.1 Setup Cluster

> Once we've confirmed that the the `~/build/kubeconfig` file exists, we can run the [cluster setup script](https://gitlab.com/opentracing-workshop/build-tools/blob/master/bin/setup-cluster). Run the following command in the Cloud Shell window: `docker run --rm -it -v "$HOME/build/kubeconfig:/root/.kube/config" registry.gitlab.com/opentracing-workshop/build-tools:latest setup-cluster`. This script will take 2-3 minutes to complete, do not close the window or interrupt the script during the execution. If you encounter an error, stop here and raise your hand.
>
> When finished executing, we should see a screen like this:

![Cluster Setup Success](/lab-02/images/img07.png)

### 2.2 Export Gitlab Credentials

> For the final task in this lab, we'll be exporting keys and tokens that Gitlab will use to configure and ship our microservice applications. The next command you'll run is: `docker run --rm -it -v "$HOME/build/kubeconfig:/root/.kube/config" registry.gitlab.com/opentracing-workshop/build-tools:latest fetch-gitlab-settings spc > gitlab-creds.txt`
> 
> When this command has executed, we need to open the file browser in Google Cloud Shell to view this file.

![Open Cloud Editor](/lab-02/images/img06.png)

> This will open a new window which allows us to open and copy the contents of the `~/gitlab-creds.txt` file easily.
>
> **IMPORTANT We cannot use cat, Vim or an in-shell editor for this step, as the copy/paste command mangles the token format.**

![Gitlab Creds](/lab-02/images/img08.png)

> ##### That's it for this lab, in [Lab 3](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#welcome-to-lab-03-gitlab-and-repository-setup) we'll be setting up Gitlab and deploying our first microservice application.
