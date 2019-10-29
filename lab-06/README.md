Welcome to Lab 06 - Observe It!
===

In this lab we will dive in and begin to understand the visualizations in Jaeger and how to break down the service to service communication within the timeline graph. In addition we'll cover several advanced use cases for microservice performance management utilizing distributed tracing technology.

Tasks:

- [ ] 1 :: [Explore Jaeger](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#explore-jaeger)
  - [ ] 1.1 :: [Finding Traces](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#11-finding-traces)
  - [ ] 1.2 :: [Understanding Traces](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#12-understanding-traces)
  - [ ] 1.3 :: [Analyzing Calls](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#13-analyzing-calls)
  - [ ] 1.4 :: [Async Processing Analysis](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#14-async-processing-analysis)
- [ ] 2 :: [Advanced Use Cases](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#advanced-use-cases-with-distributed-tracing-analytics)
  - [ ] 2.1 :: [Single Music Overview](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#21-single-music-overview)
  - [ ] 2.2 :: [Vendor Solution Overview](https://gitlab.com/opentracing-workshop/lab-notes/tree/master/lab-06#22-vendor-solution-overview)

Explore Jaeger
---

### 1.1 Finding Traces


> Open the Jaeger URL which was printed to the Google terminal shell in your browser and apply the following values to the fields on the left side nav bar:
>
> * _Yellow_ :: Set service to `owners-service`
> * _Purple_ :: Set minimum duration to `50ms`
> * _Red_ :: Increase results limit to to `200`
> * _Green_ :: Click find traces

### 1.2 Understanding Traces

We need to break down what's here in the UI before we go any further. Let's focus on the plot chart for now. We can observe frequency of requests across the Y axis, along with complexity and latency on the X axis. A larger bubble represents a call which has more complexity (calls) in the trace. We're going to click on a call which has both latency and complexity and analyze it on the next screen shot.

![Trace Lists](/lab-06/images/img01a.png)

The very first span, also called a root span, represents the entire request body. At the end of this trace the call is considered complete and the request has been closed. Now, in the advance cases there can be additional processing which occurs for async transactions. We'll cover that use case later when we look at the notification service and the use of Kafka as a pubsub provider.

Child spans are rendered within the length of the root span, their colors can help differentiate different services being called during the duration of the request. For instance, in this call, we can see teal - which represents the `customers-service` and dark brown - which represents `visits-service`. We can click the drop downs on the individual spans to investigate the endpoints that are being accessed, along with any log data collected during the transaction.

### 1.3 Analyzing Calls

![Call Analysis](/lab-06/images/img01b.png)

As we dig deeper into the calls, we can see the span for `visits-service` is abnormally long, the subsequent span shows `49.08ms` for the database response, but the entire call took 620ms. By examining the logs we can see that by the 254ms mark, the request had been prepared for the database transaction. The reason why it took so long is we're likely related to i/o warm up on the database. If this were a consistent issue, we might do some deep profiling or database troubleshooting to see if there are any missing indexes or excessive i/o as the root cause.

Distributed Tracing provides visibility into large, complex microservice environments to help you understand the behavior of your applications _in production_. If problems start manifesting, we can hopefully analyze them before they bring the entire system down.

![Deep Call Analysis](/lab-06/images/img01c.png)

### 1.4 Async Processing / Analysis

Initially, the Spring Pet Clinic microservices application didn't have a notifications service. We'll briefly cover the motivation behind introducing this new business requirement, the architecture of this new functionality, and a common occurence most software development teams will encounter when they've dedicated the time and effort it requires to implement distributed tracing in their environments.

#### What the business wants

Let's pretend for a moment that this is a real application. Notifications make sense because it can be leveraged for a number of use cases, some of which include:

* Upcoming Appointment Reminders
* Billing Payment Notifications
* Vaccination Reminders
* Courtesy / Thank You Messages

The architects decided on implementing Kafka to handle these events, since the application serves several thousand clinics and hundreds of thousands visits daily. They wanted the system to be fault tolerant, and scalable. They also decided on Kafka because it was cool (even though RabbitMQ could probably handle this use case). Does this sound familiar?

Nevertheless, the service was launched and the first notification that was implemented was a thank you message after a new visit was registered. The beta was well-recieved by the business team and they demanded it be implemented immediatly, however, the engineering team had not considered ensuring observability for the new functionality. Here is a diagram of the logic/flow of a new visit request.

```
    visit svc
  /------------\
  | POST visit | 
  \------------/
    |       |----------\
    |               producer
    \---> DB call      |
          (insert)     |    
                       |-- `create-visit-record` (topic)
                       |    VisitRecord [petId, ownerId, visitId]
                       |
                    listener
                       |
                       |  notification svc
                     /---------------------\
                     | create-visit-record |
                     \---------------------/
                       \      \      \ 
                        \      \      \-----> RestAPI call (pet)   --> DB call (select)
                         \      \-----------> RestAPI call (owner) --> DB call (select)
                          \-----------------> RestAPI call (visit) --> DB call (select)
```

As we can see above, the visit service was slightly refactored to produce a message on the `create-visit-record` topic with the IDs of the data relevant to that request. The notification service has a listener on that topic and will go fetch the values from the database and send an SMS message with pieces of data gathered from the `select` queries executed in the notication service.

Distributed tracing and the GAANT chart should visualize this request as a single transaction across multiple services with Kafka in the middle passing the message. What we see in Jaeger are two disconnected traces and their spans

![Initial Visit POST](/lab-06/images/img02a.png)

Above is the POST request initiated by submitting a new visit. All we see here is the DB call (insert), and we can see a message was produced on a topic due to the Log messages in the span - but we cannot see the trace context here.

![Notification Service Request](/lab-06/images/img02b.png)

As we can see above, the notification service made the Rest API requests to the other services, however, the originator of those request (the kafka topic being consumed) is not visualized. This disconnect may seem minor, but when operating at scale these interactions are measured and analyzed for latency, throughput and error rate. This is assuming the SRE team is using distributed tracing for [measuring SLI/SLOs](https://cloud.google.com/blog/products/gcp/sre-fundamentals-slis-slas-and-slos) and system health - which they should since that's a major benefit of observability.

Now, how do we solve this problem? Well, that's complicated, because [instrumenting Kafka with Open Tracing](https://github.com/opentracing-contrib/java-kafka-client) requires significant code level changes beyond just introducing a library.

Now, just as is the case with real world IT and business constraints, the application code which handles Kafka has yet to be instrumented because the business found a solution which automatically instruments the entire application including Kafka without any code changes. Let's take a look at how this request looks when fully instrumented in a vendored solution (Instana):

![Instana to the Rescue](/lab-06/images/img02c.png)

Advanced Use Cases (with Distributed Tracing analytics)
---

### 2.1 Single Music Overview

Now that we've covered some basics, we'll dive into some advanced use cases supplied through the Instana APM tool. The reason I'm using this tool for examples:
  
* I operate and manage a small microservice application (around 30 services) for Single Music - a small startup that I launched with two friends, and I monitor it with Instana. It's been in operation for nearly two years, and I've collected **a lot** of screenshots / use cases.
* Current Open Source technologies do not offer the deep analytic capabilities I'm about to discuss; which include aggregations, machine learning, and graph dependency mapping for contributing factor analysis.
* Our team is really small, we don't have the time or budget to maintain our own monitoring solution. 

### 2.2 Vendor Solution Overview

There are several vendors today which can provide similar functionality, and what it boils down to is your budget, technology requirements, and enterprise-ready requirements (on-premise, multi-tenant, saml, etc). I'll mention what I consider industry leaders in this space. If you're considering utilizing this technology in your stack you should be prepared to do several proof of concepts and quite a bit of research.

* [Instana](https://instana.com) <-- Caveat: I work here, and that's why it's #1 on the list
* [DataDog](https://datadog.com)
* [Honeycomb](https://honeycomb.io)
* [Elastic](https://elastic.co)
* [Lightstep](https://lightstep.com)
* [DynaTrace](https://dynatrace.com)

