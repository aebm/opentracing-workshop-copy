Welcome to Lab 05 - Monitor It!
===

In this lab we must now provide the infrastructure necessary to collect, manage, and visualize the observability data (metrics and traces) that our applications are now emitting.

Tasks:

- [ ] 1 :: [Setup Monitoring Systems](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#setup-monitoring-systems)
  - [ ] 1.1 :: [Discuss Requirements](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#11-discuss-requirements)
  - [ ] 1.2 :: [Spring Boot Admin Refresh](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#12-spring-boot-admin-refresh)
  - [ ] 1.3 :: [Deploy Monitoring Infrastructure](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#13-deploy-monitoring-infrastructure)
- [ ] 2 :: [Using Grafana](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#using-grafana)
  - [ ] 2.1 :: [Login to Grafana](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#21-login-to-grafana)
  - [ ] 2.2 :: [Add Micrometer Dashboard](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#22-add-micrometer-dashboard)
  - [ ] 2.3 :: [Add MySQL Dashboard](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#23-add-mysql-dashboard)
  - [ ] 2.4 :: [Add Custom SPC Dashboard](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#24-add-custom-spc-dashboard)
- [ ] 3 :: [Discover Alertmanager](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#discover-alertmanager)
  - [ ] 3.1 :: [Alertmanager Overview](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-05#31-alertmanager-overview)

Setup Monitoring Systems
---

### 1.1 Discuss Requirements

> In this task we will deploy and configure the following technologies:
> 
> * [Prometheus](https://prometheus.io/)
> * [Grafana](https://grafana.com/)
> * [Jaeger](https://www.jaegertracing.io/)
>
> The benefit to using the aforementioned tools is historical analytics, but it comes at the price of operational complexity. You must ensure your applications are configured and built for observability, and you must budget for the operation of the above systems - which have significant CPU/Memory/Disk requirements. We'll cover alternatives after this lab, and the benefits of using a managed solution can have when running systems that aren't proof of concept/demo environments.

### 1.2 Spring Boot Admin Refresh

> Before we get started, let's open up `Spring Boot Admin` again or refresh the page, and click on one of the application services. You'll immediatly notice a tremendous amount of more data (see screen shot below). The configuration changes we've made and libraries we introduced to our application has also enabled data to be collected automatically by `Spring Boot Admin`. There is no historical or analytic capabilities in this application, so it's only useful for "real-time" monitoring at best.

![Spring Boot Admin Refresh](/lab-05/images/img00a.png)

### 1.3 Deploy Monitoring Infrastructure

> In this step we'll go back to the Google Console and run the `setup-monitoring` command (similar to the `setup-cluster` step in [Lab 2.1](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-02#21-setup-cluster)). Open the console terminal and run this command in the terminal: 
> 
> `docker run --rm -it -v "$HOME/build/kubeconfig:/root/.kube/config" registry.gitlab.com/opentracing-workshop/build-tools setup-monitoring`
>
> At the conclusion of this command you'll see a list of URLs which should begin working within a minute or two. While you're waiting you can [view the script](https://gitlab.com/opentracing-workshop/build-tools/blob/master/bin/setup-monitoring) which installed these services. It's a fairly straight-forward setup, the complexity is baked into the helm charts - which we could spend two hours on talking about alone.

![Ingress URLs](/lab-05/images/img01.png)

Using Grafana
---

### 2.1 Login to Grafana

In this task, we'll be getting familiar with Grafana, logging in, and exploring some of the pre-installed dashboards.

![Grafana Login is not UX](/lab-05/images/img02a.png)

> Follow these steps to sign in:
> * Visit the Grafana URL provided by the installation script -
https://grafana.spc.INGRESS_IP.nip.io
> * Login to the service with the username and password: `admin:admin`

Once logged in, feel free to explore a few of the pre-installed dashboards, these were installed via the Prometheus operator.

![Kubernetes Capacity Planning](/lab-05/images/img02b.png)
![Node Stats](/lab-05/images/img02c.png)

### 2.2 Add Micrometer Dashboard

Now that we have the hang of Grafana and the unique UX, we can setup an [additional dashboards](https://grafana.com/dashboards/4701) which will instrument our Java applications running micrometer.

> Add the Micrometer dashboard:
>
> * Hover over the `+` symbol on the left nav, and click Import.
> * Input `4701` into the `Grafana.com Dashboard`, and click on any empty space on the page (this will load Options)
> * Select `prometheus` from the `Prometheus` dropdown
> * Click Import

![Import Micrometer Dashboard Pt 1](/lab-05/images/img03a.png)
![Import Micrometer Dashboard Pt 2](/lab-05/images/img03b.png)

**Optional Steps (Band-aid for Instances)**

If we click on the `Instance` drop-down, you'll notice IP addresses. These are the individual services which make up the Spring Petclinic Microservice demo. It's not very practical to have just IP addresses in here, we'd have to go look up the Pod IP every time we wanted to check our service, and those Pod IDs are ephemeral, they'll change every time we do a deployment.

We can band-aid this within Prometheus, although the fix is far from optimal (more on that later).

![Micrometer Metrics](/lab-05/images/img04a.png)

> Band-aid steps:
> 
> * Click the Settings Wheel in the upper right corner of the dashboard
> * _Teal_ :: Click `Variables` on the left settings panel
> * _Purple_ :: Click `New`

![Adding Pod Names 1](/lab-05/images/img04b.png)

> * _Red_ :: Input `pod` into the `Name` and `Label` field
> * _Purple_ :: Input `label_values(jvm_memory_used_bytes{application="$application", instance="$instance"}, pod)` into the `Query` field
> * _Teal_ :: Click `Add`

![Add Pod Names 2](/lab-05/images/img04c.png)

> * _Yellow_ :: Move the `$pod` variable under `$instance`
> * _Red_ :: Click `Save` on the left settings panel
> * _Teal_ :: Click `Back` button on the upper right corner of the dashboard

![Add Pod Names 3](/lab-05/images/img04d.png)

As we switch between `Instance` IP addresses the `pod` name will update accordingly. A more optimal solution would be to populate the `pod` name with all the instances running micrometer. This repository is open to contributions/MRs. :)

![Pod Name Added](/lab-05/images/img04e.png)

### 2.3 Add MySQL Dashboard

Remember the change we made to the MySQL helm chart values? We added support for the Prometheus operator. With that we can scrape metrics from our MySQL application. You guessed it, there is a community dashboard for that too!

![Add MySQL Dashboard](/lab-05/images/img05a.png)

> Similar to step 3, we'll be adding the community [MySQL Dashboard](https://grafana.com/dashboards/6239):
>
> * Hover over the `+` symbol on the left nav, and click Import.
> * Input `6239` into the `Grafana.com Dashboard`, and click on any empty space on the page (this will load Options)
> * Be sure to select `prometheus` in the `prometheus` drop down
> * Click Import

![MySQL Metrics Dashboard](/lab-05/images/img05b.png)

### 2.4 Add Custom SPC Dashboard

For the final dashboard, we've created a custom dashboard which will track specific metrics we configured in our application business logic.

The dashboard JSON manifest [is located here](https://gist.githubusercontent.com/notsureifkevin/104adca1ccd4b96937738f9bfbb6ba46/raw/1fddbe7eeebcb57dfb25d0407161710928d9b52f/petclinic.json)

> Paste the contents of the JSON manifest linked above:
> 
> * Hover over the `+` button on the left nav-bar, click `Import`
> * Copy the contents of the aforemented JSON file into the JSON field, then click the `Load` button
> * Choose `prometheus` as the prometheus data source
> * Click the `Import` button

![Import Petclinic Dashboard](/lab-05/images/img06a.png)

We should now see a mostly empty dashboard! This is because we haven't taken many actions within our demo application. Head over to the Spring Petclinic app, create some owners and pets, update, create visits, etc. We'll begin to see the values in this dashboard reflect those actions (this is also preparing us for the next application, Jaeger).

![Custom Petclinic Dashboard](/lab-05/images/img06b.png)

Let's move onto the final task for this lab where we uncover Alertmanager.

Discover Alertmanager
---

### 3.1 Alertmanager Overview

In this task we'll explore Alertmanager and the Prometheus console. These are stateless dashboards which derive all their settings from the Helm chart and associated configuration. In order to add/configure additional alerts, you'll need to read the documentation for the Prometheus operator. The motivation behind only allowing changes to be applied through manifests is to enforce the concept of "infrastructure as code".

> Open a browser tab and load the `Alertmanager` dashboard by copying the link provided from the Google terminal. (http://alertmanager.INGRESS_IP.io)

It's possible to configure Alertmanager and send alerts to Slack, Pagerduty, and other integrations. If we click on `Source` for any of the alerts, we'll be taken to Prometheus where we can examine the query which triggers the pre-defined alert.

If we click on `Source` for the `TargetDown` alert, we can see that there are a couple matches for this particular query. The purpose of this is to show us how these alerts are defined, via a [well-documented query language](https://prometheus.io/docs/prometheus/latest/querying/basics/) which is used to build dashboards in Grafana as well.

![Alertmanager](/lab-05/images/img07a.png)
![Prometheus](/lab-05/images/img07b.png)

---

In [Lab 6](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#welcome-to-lab-06-observe-it), the final lab of this series, we'll investigate distributed tracing with Jaeger and discuss advanced analytics capabilities which can be derived from data gathered via distributed tracing.