Welcome to Lab 03 - Gitlab and Repository Setup
===

In this lab we'll be setting up Gitlab, copying and configuring our repository, also we'll setup the CI/CD process to deploy to our newly configured K8S cluster in GKE.

## Tasks

- [ ] 1 :: [Setup Application Repository](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#setup-application-repository)
  - [ ] 1.1 :: [Sign Up / Login to Gitlab](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#11-sign-up-login-to-gitlab)
  - [ ] 1.2 :: [Create New Project](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#12-create-new-project)
- [ ] 2 :: [Configure CI/CD](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#configure-cicd)
  - [ ] 2.1 :: [Setup Environment Variables](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#21-setup-environment-variables)
  - [ ] 2.2 :: [Configure Kubernetes Integration](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#22-configure-kubernetes-integration)
  - [ ] 2.3 :: [Run Pipeline Validation](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#23-run-pipeline-validation)

Setup Application Repository
---

### 1.1 Sign Up / Login to Gitlab

> To get started, you'll need to either sign into Gitlab with your account, or create a new account. We're going to create a new account for those here who may be new to Gitlab. You may need to help train Google's AI - especially if the entire room starts creating accounts. Be sure to use a valid email address because you'll need to confirm it in order to use your account. You may go to the next step if you already have an account.

![Sign Up for Gitlab Account](/lab-03/images/img01.png)

### 1.2 Create New Project

![Create New Project](/lab-03/images/img02.png)

> Click the `Import Project` tab, and click `Repo by URL`. You will be cloning the `spring-petclinic-kubernetes` repository next.

![Import Project](/lab-03/images/img03.png)

> Fill out the following values:
>
> * Git Repository URL: `https://github.com/notsureifkevin/spring-petclinic-kubernetes.git`
> * Project Name: `spring-petclinic-kubernetes`
> * Set the Project to `Public`
> * Click `Create Project`

![Create Project](/lab-03/images/img04-v2.png)

Configure CI/CD
---

### 2.1 Setup Environment Variables

> First we must populate the Ingress IP as a build time variable - this will configure our applications to respond to requests from the outside and route accordingly. You will find this IP address in the `gitlab-creds` file you exported and saved earlier. The `Ingress IP:` field is near the bottom of this document. If there is no IP address there, you can find the IP address in the google console by clicking on `Services`. An example can be found below:

![Service Ingress IP](/lab-03/images/img05a.png)

> In the above example the Ingress IP address is surrounded by a green box. **DO NOT USE THE IP ADDRESS IN THE ABOVE EXAMPLE**
> 
> In the following example, you may need to scroll down to find `Settings`:
> 
> * Click `Settings`
> * Click `CI/CD`

![CI/CD Settings](/lab-03/images/img05b-v2.png)

> * Click `Expand` next to `Variables`
> * Fill in `INGRESS_IP` and the value you've collected from either `gitlab-creds` or from the Service Ingress IP in the Google Console.
> * Click `Save variables`

![Input Ingress IP](/lab-03/images/img05c.png)

## 2.2 Configure Kubernetes Integration

> In the next step you will be configuring the Kubernetes Integration in Gitlab by providing the other values which are exposed in the `gitlab-creds` document
> 
> * Click `Operations` and the `Kubernetes` Sub-menu item on the left nav-bar.
> * Click `Add Kubernetes Cluster`

![Kubernetes Ops](/lab-03/images/img06a.png)

![Add Kubernetes](/lab-03/images/img06b.png)

> Before filling in these values, be sure to click the `Add existing cluster` tab
>
> The values for these fields can be found in `gitlab-creds` which was created earlier in Lab 02:
> 
> * Kubernetes Cluster Name: `my-cluster`
> * Environment Scope: `*`
> * API URL: `<Copy URL from gitlab-creds>`
> * CA Certificate: `<Copy CA CERT from gitlab-creds>`
> * Token: `<Copy Token from bottom of gitlab-creds>`
> * RBAC-enabled cluster: `checked`
> * Gitlab-Managed cluster: `UNCHECK`
> * Project Namespace: `spc`
> 
> Once you've populated these values, click `Add Kubernetes Cluster`, you'll validate these entries in the next step during > our first build.

![For real - Add Kubernetes](/lab-03/images/img06c-v2.png)

### 2.3 Run Pipeline Validation

> The following next steps will validate your pipeline is configured correctly and execute your initial builds
>
> * Click `CI/CD` in the left-nav menu, then click on `Run Pipeline`
> * Click `Run Pipeline` for the `master` branch/tag
> * Once the pipelines has begun executing, click the `Play` button next to `validate-k8s`
> * To see this job run in real-time, click the `validate-k8s` button
>
> The images below will detail each step:

> * Click `CI/CD` in the left-nav menu, then click on `Run Pipeline`

![Run Pipeline](/lab-03/images/img07a.png)

> * Click `Run Pipeline` for the `master` branch/tag

![Run Pipeline For Real This Time](/lab-03/images/img07a1.png)

> * Once the pipelines has begun executing, click the `Play` button next to `validate-k8s`

![Run Validate Job](/lab-03/images/img07b.png)

> * To see this job run in real-time, click the `validate-k8s` button

![See Validate Job Run](/lab-03/images/img07c.png)

> This job should complete and be successful. If you do not see the `INGRESS_IP` value, and the tiller pod listed from `kubectl get pods` **STOP HERE** and raise your hand before moving on to [Lab 4](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#welcome-to-lab-04-deploy-all-the-things).