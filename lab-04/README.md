Welcome to Lab 04 - Deploy All The Things!
===

In this lab, we will deploy our microservice application and explore some of the tooling available to use to monitor and manage our application out of the box.

## Tasks

- [ ] 1 :: [Deploy App / Infrastructure Services](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#deploy-app-infrastructure-services)
  - [ ] 1.1 :: [Deploy Service Dependencies](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#11-deploy-service-dependencies)
  - [ ] 1.2 :: [Deploy Microservice Applications](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#12-deploy-microservice-applications)
  - [ ] 1.3 :: [Test Drive the Pet Clinic](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#13-test-drive-the-pet-clinic)
- [ ] 2 :: [Merge / Deploy Observable Application](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#merge-deploy-observable-application)
  - [ ] 2.1 :: [Discuss Changes and Requirements](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#21-discuss-changes-and-requirements)
  - [ ] 2.2 :: [Create Merge Request](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#22-create-merge-request)
  - [ ] 2.3 :: [Code Review](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#23-code-review)
  - [ ] 2.4 :: [Deploy Changes](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-04#24-deploy-changes)

Deploy App / Infrastructure Services
---

### 1.1 Deploy Service Dependencies

> In order for our application to work, we must deploy configuration which will instruct our ingress proxy [traefik](traefik.com) to route requests at our service endpoints. This script will also deploy the backend services such as Kafka and MySQL for our application. You can see the script and resources being deployed here:
 
* Deploy Dependency Script: https://gitlab.com/opentracing-workshop/spring-petclinic-kubernetes/blob/master/scripts/deploy_backend.sh
* Ingress Rules: https://gitlab.com/kcdockerio/spring-petclinic-kubernetes/blob/master/helm/spring-petclinic-ingress-rules/templates/traefik-ingress.yaml
* MySQL Configuration: https://gitlab.com/opentracing-workshop/spring-petclinic-kubernetes/blob/master/helm/spring-petclinic-database-server/values.yaml
* Kafka Configuration: https://gitlab.com/opentracing-workshop/spring-petclinic-kubernetes/blob/master/helm/spring-petclinic-kafka-server/values.yaml

> We can trigger the `deploy_backend.sh` script by clicking on `CI/CD -> Pipelines` in the left nav, click on the `>>` button, and click the _Play_ button next to `deploy-dependencies`. If we click the same bubble again, we can watch the job run in real time by clicking on `deploy-dependencies`. Navigating / Using the CI/CD functionality can be a bit clunky at first pass, but once you've clicked around a few things it should become familiar enough.

![Deploy Dependencies](/lab-04/images/img01.png)

### 1.2 Deploy Microservice Applications

> Once the job is complete and successful, we are ready to deploy our microservice application. This next step will execute a series of actions which will deploy the Helm chart for the Spring Petclinic services. 

* Deploy Service Script: https://gitlab.com/opentracing-workshop/spring-petclinic-kubernetes/blob/master/scripts/deploy_services.sh
* Spring Petclinic Helm Chart: https://gitlab.com/opentracing-workshop/spring-petclinic-kubernetes/tree/master/helm/spring-petclinic-kubernetes

> During the build process, we generated a `modules.info` file from `pom.xml`, which is used by the deploy script to determine the services to deploy via Helm.
> 
> Similar to how we deployed the Dependencies, we're going to Deploy the application this time. Click the last bubble in the stages column, and then click the _Play_ button next to `deploy-services`. You can watch this job run in real time by clicking on `deploy-services` after you've clicked `Play`. This job will take a few minutes to run, but once it's done move onto the next step!

![Deploy Services](/lab-04/images/img02.png)

### 1.3 Test Drive the Pet Clinic

> We can now see our deployment in GKE, so switch back to the Google Console and start investigating your application. From here, you can inspect your application services, metrics, and logs. You can even load your application in your browser and begin interacting with it! *NOTE* Be sure to replace `INGRESS_IP` with the Ingress IP address you saved from `gitlab-creds` earlier.

* Visit http://admin.INGRESS_IP.nip.io for the Spring Boot Admin control panel.
* Visit http://spc.INGRESS_IP.nip.io for the application itself!

![GKE Has Services](/lab-04/images/img03a.png)
![Spring Boot Admin](/lab-04/images/img03b.png)
![Spring Boot Admin 2](/lab-04/images/img03d.png)
![Spring Pet Clinic](/lab-04/images/img03c.png)

> Feel free to spend some time here and please ask questions! We just completed roughly two weeks worth of integration work in less than an hour, there is a lot to ingest here. Once you've soaked in the awesomeness, let's check out what it's going to take to add monitoring and tracing to our application.

Merge / Deploy Observable Application
---

### 2.1 Discuss Changes and Requirements

> We're going to create a new merge request in Gitlab and deploy changes which will begin exporting metrics with [Micrometer](https://micrometer.io/) and distributed tracing data with [OpenTracing](https://opentracing.io/).
> 
> Micrometer provides one important aspects of modern application observability, which is the ability to export the runtime characteristics of our application such as memory and CPU usage, along with business specific metrics such as Requests Per Second and custom metrics we can implement within our applications. We'll use a tool called [Prometheus](https://prometheus.io/) to collect those metrics, and another tool called [Grafana](https://grafana.com/) to visualize them.
> 
> Opentracing provides an additional capability, which is exporting the timing and metadata around the requests which are being served by the services within your microservice application. This data can be used to quickly determine if a particular service is taking longer than expected to respond to requests. There are other use-cases for this data, such as analyzing diffs between requests and drilling down into the performance characteristics for a given user or entry point. We can also analyze the performance of 3rd party systems and APIs to determine the impact they are having on our own services. We'll be utilizing [Jaeger](https://www.jaegertracing.io/) to analyze the data exported from our applications.

### 2.2 Create Merge Request

> Let's create the merge request, so head back to Gitlab and follow these steps:

* Click `Repository` on the left-nav, then click `Compare`
* Change `Source` to `add-oss-monitoring`
* Click `Create merge request`
* Leave all settings at their default, scroll and click `Submit merge request`

![Create Merge Request](/lab-04/images/img04.png)
![Submit Merge Request](/lab-04/images/img04a.png)
![Ship It](/lab-04/images/img06.png)

> Merge the observability changes! We will be reviewing the code as these changes are being built and pushed to the docker repository. Click `Merge`

### 2.3 Code Review

> Click on the `Changes (n)` tab to reveal the modifications we're making to our microservice application. There are a number of duplicated changes we won't be covering, but we will go into detail on each unique change and explain what we're doing to enable our applications for observability.
> 
> The first change you should see is to the `helm/spring-petclinic-database-server/values.yaml` file. This file contains the configuration for the `mysql` helm chart deployment. Let's examine the changes.

* a. [Helm Repository - MySQL](https://github.com/helm/charts/tree/master/stable/mysql) - We're configuring MySQL to run a [sidecar pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) that will collect metrics from MySQL and export them via a prometheus endpoint.
* b. This is the magic sauce in the [prometheus operator](https://github.com/coreos/prometheus-operator). This annotation tells prometheus that it should collect metrics from this service.

![MySQL Helm Review](/lab-04/images/img05a.png)

> The next change we'll be reviewing is to the `helm/spring-petclinic-kubernetes/templates/deployment.yaml` manifest. This manifest is used across all the spring petclinic microservices, which is the real power behind helm since it can be used to reduce config duplication.
> 
> You can see below that we're adding multiple environment values to the template, which will be applied to each service so long `jaeger.enabled == true` in the corresponding service `values.yaml` file (we'll see that later in the review). Now, within the last month or two the Jaeger Tracing operator has matured quite a bit and it's worth looking at that pattern (similiar to prometheus) for deployment. Within the next month or two this workshop will be updated to utilize that technology. For now, let's break each of these variables:

* `SPRING_PROFILES_ACTIVE == jaegertracing` - This setting enables our bootstrap method in our services, and indicates we're using the jaeger client to export trace spans and logs
* `JAEGER_ENDPOINT == jaegerUrl` - This is the internal endpoint which is used by the jaeger client within our services to send data
* `JAEGER_SAMPLER_TYPE == const` - We'll collect every trace for now, for production systems that handle thousands of requests per minute (or more) this will likely cause issues due to the overhead of this implementation
* `JAEGER_SAMPLER_PARAM == 1` - This would be set to a more reasonable number such as `1000` in high-load production systems, which would mean one out of every thousand requests would be recorded
* `JAEGER_REPORTER_LOG_SPANS == false` - Logs that are emitted during an instrumented span will be logged within the trace. This is great for troubleshooting but adds a lot of overhead.
* `JAEGER_SERVICE_NAME == {{ .Release.Name }}` - Populates the service name value from the Helm service template

![Spring Petclinic Helm Chart Review](/lab-04/images/img05b.png)

> If you're not familiar with Java and Maven projects, `pom.xml` is a manifest file for the project, it includes dependencies, libraries, and other configuration details that are required to compile our micro service applications. In the following change, we've added multiple dependencies which are required to export metrics and tracing data. Let's break down each dependency:

* a. [Micrometer](https://micrometer.io/) - Collects runtime metrics both automatically and manually. Since this is a spring boot application, it will collect metrics such as the number of incoming requests, response status codes (200, 404, etc), and library specific metrics such as RabbitMQ and the number of messages processed, or JDBC and the number of queries executed. You can also instrument specific methods to extract timing details, counts, and other business-like metrics.
* b. [Micrometer Prometheus Registry](https://github.com/micrometer-metrics/micrometer/tree/master/implementations) - Exports the data collected by Micrometer into a format which can be ingested by Prometheus. There are other registry formats such as Influx, Graphite, and many others.
* c. [Jaeger Tracing Client](https://github.com/jaegertracing/jaeger-client-java) - Responsible for shipping trace spans and logs to the Jaeger service
* d. [Spring Cloud Opentracing](https://github.com/opentracing-contrib/java-spring-cloud) - Automatically wraps specific technologies within Spring Boot such as RestTemplate, JDBC, JMS, etc. This is required to capture timing details and meta data around transactions between other services and data systems such as MySQL, Oracle, RabbitMQ. There are dozens of technologies supported within this library - as long as you're using standard approaches to handling requests and data you shouldn't have to worry about manual instrumentation to gather trace context.

![Spring Petclinic Dependencies](/lab-04/images/img05c.png)

> Once our project has the necessary dependencies we must configure the application to expose those metrics for collection. In this next image we're adding the necessary configuration for Micrometer and Prometheus. This is required for each application we want metrics exposed.

![Metric Configuration](/lab-04/images/img05d.png)

> We must also bootstrap Opentracing within each application that we want to trace:

![Opentracing Configuration](/lab-04/images/img05e.png)

> Finally, if we want to collect custom business metrics, we must implement the logic to emit those metrics when requests are made within the application.

![Custom Metrics](/lab-04/images/img05f.png)

> Now that we've reviewed our code, it's time to merge!

### 2.4 Deploy Changes

> In order to apply the changes we've made to both the database and the services, we must run both deployment tasks. Once the `Build` and `Push` tasks have passed, run both the Deploy tasks.

![Deploy It](/lab-04/images/img07.png)

> In the [next lab](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#welcome-to-lab-05-monitor-it) we'll be deploying the necessary infrastructure to collect, manage, and visualize the telemetry and contextual data our applications are now emitting.



