# Observability Workshop: Kubernetes, Tracing, and Metrics

This repository holds labs which can be delivered as a self-paced or instructor-led workshop. You can read more about this workshop in the summary below.

Time Requirements:

- w/ Audience Participation: 90-180 minutes (depending on class size/time constraints)
- Instructor Only: 60-90 minutes

---

## Lab Overview

- [Lab 01 - Signing Up for Google Cloud](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-01#welcome-to-lab-01-google-cloud-sign-up)
- [Lab 02 - Setting Up Kubernetes in Google Cloud](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#welcome-to-lab-02-kubernetes-setup)
- [Lab 03 - Setting up Gitlab and our Microservice Application Repository Kubernetes Integration](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-03#welcome-to-lab-03-gitlab-and-repository-setup)
- [Lab 04 - Deploying our Microservice Application and Adding Observability](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#welcome-to-lab-04-deploy-all-the-things)
- [Lab 05 - Monitoring Application Metrics with Grafana / Prometheus](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#welcome-to-lab-05-monitor-it)
- [Lab 06 - Observing with Jaeger and Advanced Distributed Tracing Use Cases](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#welcome-to-lab-06-observe-it)

## Workshop Summary

The goal of this workshop is to familiarize application and site reliability engineers with the benefits that modern observability tools provide to the builders and curators of cloud native applications.

In this workshop attendees will configure a RBAC-enabled vanilla K8S cluster in GKE, deploy Prometheus and Jaeger in support of observing and monitoring a distributed microservice application, instrument that application by introducing libraries and tooling for capturing metrics and distributed traces. Each attendee will configure, update, and deploy a cloud native application utilizing the following technologies:

* Google Kubernetes Engine - we will bootstrap a small K8S cluster using Googles free $300 trial.
* Gitlab - we will implement a CI/CD pipeline with our cloud native application in order to build and deploy our application while we integrate Prometheus, Opentracing and Jaeger into our application.
* Traefik - we will leverage a popular alternative to nginx for handling routing and ingress for our applications.
* Helm - we will utilize Helm to deploy policies, monitoring services, ingress rules, operators, and our demo application.
* Spring Petclinic Microservices - we will utilize a distributed Java/Springboot/Angular based cloud native application. We will start with an unmonitored version and work together to bring observability to our application.
* Grafana / Prometheus - we will demonstrate how to utilize prometheus operators to configure prometheus scrape targets. We will utilize micrometer to expose JMX metrics, install custom dashboards, and setup custom business metric dashboards and alerting rules.
* Jaeger / Opentracing - we will demonstrate how to utilize the Jaeger client and opentracing libraries to gain visibility into the communication happening between your microservices and backend systems.

During this workshop we will discuss and explore some of the following topics:

* How Distributed Tracing works and why it's become a requirement when deploying cloud native applications.
* The pitfalls, challenges, and solutions to managing and deploying observability at scale.
* What's missing? Important aspects of our operations are't being monitored, let's find out what and, more importantly, how to fix it.
* The history of Spring Petclinic and the decision behind refactoring to no longer depend on Eureka/Zuul (Spring Cloud).
* Challenges of utilizing Opentracing implementation in non-java languages/frameworks along with some code samples.
* Where is the innovation happening in observability? Let's discuss automation, machine learning, and the future.

Technical Requirements

* Modern unix-like OS and a browser (windows should work, if you have git bash)
* git, code editor, openssh, vim, bash
  * basic understanding and recent experience using these tools will be beneficial when participating with the group during the lab sessions
* google cloud account with or without billing enabled*
* gitlab account**

> \* : Google offers a $300 voucher to use for their trial. However, Google Cloud requires a credit card when signing up to verify you're not a bitcoin miner. Even if you have an account I recommend creating a new gmail account (if necessary) and using that.
> \** : Gitlab is free. You'll be forking a repository and pushing code/configuration changes to that fork.

Developer Requirements

* Some programming experience [java, c, php, python, etc]
* Development tooling experience [make, mvn, yaml, vim]
* Unix-like tooling [bash, scripting, vim]
* Basic networking (DNS/routing)
  * We will use `nip.io` to give our application a public wildcard domain. If you manage your own domains you could alternatively use your own domain name (we will only cover using `nip.io` in the workshop)

Q+A:

> Q: Will we walk through signing up for Google Cloud and Gitlab at the beginning of the workshop?

Yes. However, I highly recommend you do this prior to the workshop. Google `new gitlab account` and `google cloud trial` for answers.

> Q: Why Gitlab / Google?

Gitlab has a built-in CI/CD pipeline system and integration with K8S tooling. Google has an extremely mature and feature-rich managed solution for k8s. Also, it's free.

> Q: I already have a "company", "personal", "sandbox" Google Cloud account, can I just use that?

This would not be recommended. Your account could have policies or restrictions which would prevent you from following along with the exercise.

> Q: I got this, I know more about Google Cloud than Google does. How much will this cost me?

Free if you just sign up for a new account. Otherwise, it's about $10-15 for the compute/network/etc time over the course of the workshop.

> Q: Why can't we just use minikube? 

This workshop was built using a 2018 XPS 13 w/ a quad core and 16gb ram using minikube. The project quickly outgrew this machine and operationally requires at minumum of 8 real cores and 20gb of ram (at load).

> Q: Wow, your laptop sucks. I have a 12-core alienware with 64gb of ram. Can I just use minikube?

The project itself includes dependencies on large frameworks and libraries, requires heavy weight compilers, and multiple build tools. By operationalizing this project, we offload all of the network and compute requirements to the cloud - we only need our local machines to make code changes and commit them.

To answer your question, yes, you can. But please, don't.