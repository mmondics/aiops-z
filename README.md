# AIOps with OpenShift on IBM zSystems and LinuxONE

In this tutorial, you will walk become familiar with IBM's three strategic AIOps solutions - Instana, Turbonomic, and IBM Cloud Pak for Watson AIOps - and the capabilities they have to monitor and manage OpenShift on IBM zSystems.

## Pre-requisites

1. Access to an OpenShift on IBM zSystems cluster with the Robot Shop application
2. Access to an Instana instance that is observing the OpenShift on IBM zSystems cluster as well as the Robot Shop microservices
3. Access to a Turbonomic instance that is configured with both OpenShift and Instana as managed targets
4. Access to an IBM Cloud Pak for Watson AIOps cluster that is configured with OpenShift, Instana, and Turbonomic as connections

## AIOps Overview

## Environment Overview

![aiops-arch](aiops-arch.drawio.svg)

## Exploring the Robot Shop Sample Application

1. **Open a web browser such as Firefox.**

2. In the browser, **navigate to your OpenShift console.** 

    The OpenShift console typically begins with `https://console-openshift-console-`. Reach out to your OpenShift administrator if you do not have this address.

    You will now see the OpenShift console login page.

    ![openshift-console-login](/images/openshift-console-login.png)

3. **Log in with your OpenShift credentials.**

4. Under the developer perspective, navigate to the Topology page for the `robot-shop` project.

    ![robot-shop-topology](images/robot-shop-topology.png)

    Robot Shop is a simulated online store where you can purchase robots and AI solutions. Robot Shop is a polyglot - meaning it is made up of many different microservices that were written in different programming languages. There are 12 different microservices written in languages including NodeJS, Python, Spring Boot, and Go, along with containerized databases including MongoDB, MySQL, and Redis. Each icon in the Topology represents an OpenShift deployment for a specific microservice. Each microservice is responsible for a single function of the Robot Shop application that you will see in the following steps.

5. Open the Robot Shop web application by clicking the small button in the top right of the `web` icon.

    This is simply a hyperlink that will take you to the Robot Shop application homepage.

    ![web-route](images/web-route.png)

    The Robot Shop homepage should appear like the screenshot below, with all of the same options in the left side menu.

    ![robot-shop-1](images/robot-shop-1.png)

6. Explore the website and its functionalities from the left side menu.
   
    Note: don't skip past this step - navigating through the website will generate some pretty cool insights in the Instana website monitoring section.

    You can register a new user, explore the catalog of purchasable robots, give them ratings, and simulate a purchase.

    Notice that from the OpenShift and Robot Shop perspectives, you don't get much of a sense of how the various microservice applications are plumbed together, how they are performing, if they have the correct amount of resources, or if any issues are affecting the application currently or in the near future. In other words, there is a lack of *observability*, *application performance management*, and *proactive problem remediation*.

## Instana

### Overview of Instana Observability

Instana is an enterprise observability solution that offers application performance management - no matter where the application or infrastructure resides. Instana can monitor both containerized and traditional applications, various infrastructure types including OpenShift, public clouds & other containerization platforms, native Linux, z/OS, websites, databases, and more. The current list of supported technologies can be found [in the Instana documentation](https://www.ibm.com/docs/en/obi/current?topic=supported-technologies).

For this tutorial, we have set up an environment that includes Instana running on a Linux server which is monitoring an OpenShift on an IBM zSystems cluster through an Instana agent which you will look at in the next section.

### Viewing the Instana Agent on OpenShift

7. In the OpenShift cluster, navigate to the `instana-agent` project then click the circular icon in the Topology page.

    ![instana-agent-daemonset](images/instana-agent-daemonset.png)

    This is the Instana agent that is collection all the information about OpenShift and the containerized applications running on it and then sending that information over to the Instana server.
    
    The agent is deployed as a [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), which is a Kubernetes object that ensures one copy of the pod runs on each compute node in the cluster. Each individual Instana agent pod is responsible for the data and metrics collection for the compute node they run on.

8. Click the `view logs` hyperlink next to one of the pods in the right-side menu.

    ```text
    2023-03-01T17:13:46.589+00:00 | INFO | stana-agent-scheduler-thread-9-1 | turesManagerImpl | com.instana.agent-bootstrap - 1.2.25 | Installed instana-kubernetes-sensor
    2023-03-01T17:13:46.593+00:00 | WARN | stana-agent-scheduler-thread-9-1 | yDiscoveryTicker | com.instana.agent - 1.1.677 | Discovery for com.instana.plugin.kubernetes took too long (28512 ms)
    2023-03-01T17:13:46.608+00:00 | INFO | 7c9-865d-446b-8ad9-ada0c5fb7a59) | Kubernetes | com.instana.sensor-kubernetes - 1.2.139 | Activating Kubernetes Sensor
    2023-03-01T17:13:46.662+00:00 | INFO | 7c9-865d-446b-8ad9-ada0c5fb7a59) | Kubernetes | com.instana.sensor-kubernetes - 1.2.139 | Using master url https://172.30.0.1:443/
    2023-03-01T17:13:47.152+00:00 | INFO | 7c9-865d-446b-8ad9-ada0c5fb7a59) | Kubernetes | com.instana.sensor-kubernetes - 1.2.139 | Subscribed to leadership events
    2023-03-01T17:13:47.997+00:00 | INFO | tana-sensor-scheduler-thread-2-2 | ableResourceInfo | com.instana.sensor-kubernetes - 1.2.139 | Start watching pods on node compute-0.atsocpd1.dmz...
    2023-03-01T17:13:50.950+00:00 | INFO | stana-agent-scheduler-thread-9-1 | turesManagerImpl | com.instana.agent-bootstrap - 1.2.25 | Installed instana-host-sensor
    2023-03-01T17:13:51.080+00:00 | INFO | 74c-6ecc-49c2-995a-e57f52e12cdb) | Host | com.instana.sensor-host - 1.1.147 | Activated Sensor
    2023-03-01T17:13:53.975+00:00 | INFO | stana-agent-scheduler-thread-9-1 | turesManagerImpl | com.instana.agent-bootstrap - 1.2.25 | Installed instana-gcp-services-discovery
    2023-03-01T17:13:53.984+00:00 | WARN | stana-agent-scheduler-thread-9-1 | yDiscoveryTicker | com.instana.agent - 1.1.677 | Discovery time (58652 ms)
    2023-03-01T17:14:15.061+00:00 | INFO | c66-b1ef-4bc0-bc2a-b810abc8cdd3) | Crio | com.instana.sensor-crio - 1.0.10 | Activated Sensor 7f31b2cbe57b25720a4faf38ca86003fbd940c86d56cf02f78ed752d5e409dcd
    2023-03-01T17:14:15.105+00:00 | INFO | 8f9-dbcf-4c46-b04d-f396b7f048a2) | Crio | com.instana.sensor-crio - 1.0.10 | Activated Sensor 5aac7fe96187321e5302478dbfc64feda35d108e59b6c3dc0c42f094435ecd7b
    2023-03-01T17:14:15.110+00:00 | INFO | 61d-5569-47e8-9196-8128bb8944c5) | Crio | com.instana.sensor-crio - 1.0.10 | Activated Sensor 7034d4698290b57385bcc5a85fca3faa4fd0605a24046ac15b4bd6c9a436813a
    2023-03-01T17:14:15.115+00:00 | INFO | c3b-a47a-4856-acef-c8a8b8d3882a) | Crio | com.instana.sensor-crio - 1.0.10 | Activated Sensor 3dbe00e2b1da3d427deaaa9321ef3ad88ba5fecd6eac47483da89e4a50375ab5
    2023-03-01T17:14:15.207+00:00 | INFO | 0c7-c787-45c4-912d-dd8f93c24d66) | Crio | com.instana.sensor-crio - 1.0.10 | Activated Sensor e49ed98a6c4cf6271d009488d9292858912602915af4be8772664d24d4f36f68
    2023-03-01T17:14:15.253+00:00 | INFO | ab0-768f-42e8-a27b-96f4f149b22d) | Crio | com.instana.sensor-crio - 1.0.10 | Activated Sensor 3268dbca7f621f728066d2e0d73bc5cdc803f3ea218e49d997bcb4519f2058df
    ```

    If you would like to see what data the agents are collecting or if you need to debug issues with collecting data from certain workloads running on OpenShift, these pod logs are a good first place to look.

    In the next section, you will take a look at the other side of the OpenShift Instana agent - the Instana server that is receiving the data.

### Navigating the Instana Dashboard

9. In a web browser, navigate to your Instana server. Reach out to your Instana administrator if you do not have this address.

10. Log in with your Instana credentials.

    When you first log into Instana, you will be taken to the Home Page. This is a customizable summary page that shows the key metrics for any component of your environment in the timeframe specified in the top right of the screen.

    ![instana-timeframe](https://raw.githubusercontent.com/mmondics/media/main/images/instana-timeframe.png)

11. Set the time frame to Last Hour and click the Live button.

    You are now seeing metrics for our environment over the previous hour, and it is updating in real time.

    The next thing to notice is the menu on the left side of the page. If you hover over the left side panel, you will see a menu of links that will let you dive into different sections of the Instana dashboard, rather than seeing every option.

12. Click the left side panel so the menu appears.

    ![instana-menu](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu.png)

    Next, you will go through each section in the menu, starting with Websites & Mobile Apps.

13. Click the Websites & Mobile Apps option.

    ![instana-menu-websites](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu-websites.png)

    You can see that Instana is monitoring one website named Robot Shop Website. This is the set of webpages associated with the Robot Shop sample application we have deployed on OpenShift on IBM zSystems. Instana supports website monitoring by analyzing actual browser request times and route loading times. It allows detailed insights into the web browsing experience of users, and deep visibility into application call paths. The Instana website monitoring solution works by using a lightweight JavaScript agent, which is embedded into the monitored website.

14. Click the Robot Shop Website hyperlink.

    ![instana-website-metrics](https://raw.githubusercontent.com/mmondics/media/main/images/instana-website-metrics.png)

    If the page is empty, navigate back to the Robot Shop website from the OpenShift topology page and click around the site to generate data.

    After interacting with the webpages, we now see metrics for page loads and transitions, loading times, any errors, and many more performance indicators. Within this website, there are all kinds of filters and tabs with more information.

15. Navigate through the various tabs to show more data - Speed, Resources, HTTP requests, and Pages.

    What we've seen so far is all related to the *website* metrics, not the *application* itself. In the next section, we will dig into how the application running on OpenShift on IBM zSystems is running.

16. Click the Applications option in the left side menu.

    ![instana-menu-applications](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu-applications.png)

17. Click the Robot Shop Z Application hyperlink.

    An application perspective represents a set of services and endpoints that are defined by a shared context and is declared using tags. For example, in this tutorial, the Robot Shop Z application perspective encompasses all services and endpoints that meet the tag `kubernetes-namespace=robot-shop`. Because all of our containerized microservices are running in an OpenShift namespace named `robot-shop`, they all appear in this application perspective. This is a very simple example of a tag, and it works because we only have one Kubernetes (OpenShift) cluster, so only one namespace named `robot-shop`. If we had another cluster with a namespace of the same name, we might want to add another tag to select the Kubernetes cluster's zone or name.

    Alongside the Robot Shop application running in OpenShift, there is a container running a Python application that generates load to each microservice. The metrics you see now in the application perspective are coming from that load generator. At the top of the page, you can see the total number of calls, the number of erroneous calls, and the mean latency for each call over the past hour. Based on the number of total calls, you can see that Instana is able to monitor an incredible amount of data in real time and provide actionable insights against it. Each chart can be interacted with in various ways. For example, the total calls graph can be filtered by return code.

18. Click to hide each return code other than `500`.

    ![instana-application-codes](https://raw.githubusercontent.com/mmondics/media/main/images/instana-application-codes.png)

    As with the Websites & Mobile Applications section, the Application perspective has various tabs that contain different information. When it comes to microservices, one of the most helpful tabs in the Application perspective is the Dependency graph.

19. Click the Dependencies tab.

    ![instana-dependencies](https://raw.githubusercontent.com/mmondics/media/main/images/instana-dependencies.png)

20. The dependency graph offers a visualization of each service in the Application perspective, which services interact with each other, and also some visual representations of errors, high latency, or erroneous calls. You can select these options in the top left of the graph.

21. Click the dropdown labeled None and select Max Latency.

    ![instana-dependency-graph](https://raw.githubusercontent.com/mmondics/media/main/gifs/instana-dependency-graph.gif)

    You can now see that the cities microservice is the largest contributor to application latency. There are more tabs in the Application perspective related to each individual service, error and log messages, the infrastructure stack related to the tag we specified, and options to configure the Application perspective.

22. Click through each of the tabs.

    If you pay attention while clicking through these tabs, you will notice that the payments service has an unusually high number of erroneous calls, but we'll get to that in a later section.

23. Click the Platforms -> Kubernetes option in the left side menu.

    ![instana-menu-kubernetes](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu-kubernetes.png)

24. Click the openshift-z (cluster) hyperlink.

    OpenShift and other Kubernetes clusters are monitored by simply deploying a containerized Instana agent onto the cluster. Once deployed, the agent will report detailed data about the cluster and the resources deployed into it. Instana automatically discovers and monitors clusters, CronJobs, Nodes, Namespaces, Deployments, DaemonSets, StatefulSets, Services, and Pods.

    The Summary page shows the most relevant information for the cluster as a whole. The CPU, Memory, and Pod usage information are shown. The other sections, such as "Top Nodes" and "Top Pods" show potential hotspots which you might want to have a look at.

25. Click the Details tab.

    The Details tab displays information about the health of critical components of the Kubernetes cluster, such as the controller, scheduler, and etcd.

26. Click the Events tab.

    The Events tab shows all of the Kubernetes events in the cluster in real time. It also provides links to the related objects for quick access. The remaining tabs are all Kubernetes objects in the cluster and the metrics displayed depend on the object type.

27. Click through the tabs to show various objects and their metrics.

    Instana also supports monitoring of Cloud Foundry and VMWare Tanzu clusters, but that is outside the scope of this demonstration.

28. Click the Infrastructure option in the left side menu.

    ![instana-menu-infrastructure](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu-infrastructure.png)

    You will be taken to the Infrastructure map. The Infrastructure map provides an overview of all monitored systems, which are initially grouped by options of your choice. Within each group are pillars comprised of opaque blocks. Each pillar as a whole represents one agent running on the respective system. Each block within a pillar represents the software components running on that system and will change color to reflect the component's health.

29. With the filters in the bottom right of the page, select the Host Perspective and OS architecture for Grouping.

    ![instana-infrastructure-1](https://raw.githubusercontent.com/mmondics/media/main/images/instana-infrastructure-1.png)

    Now we are looking at the three monitored systems grouped by architecture. Each block within the pillars is a single process or microservice. Let's take a different look at the OpenShift cluster.

30. Select the Container Perspective and Grouping by Kubernetes Namespace.

    ![instana-infrastructure-2](https://raw.githubusercontent.com/mmondics/media/main/images/instana-infrastructure-2.png)

    This is a better view of the containerized processes running in our monitored environment. As you might be able to tell by the color coding of the different processes or microservices, we have a few errors with the Robot Shop application. We'll come back to this page in a later section to debug.

31. Click the Analytics option in the left side menu.

    Instana includes a very powerful AI-driven analytics solution out of the box. These analytics are integrated into each of the panels we've looked at so far, but you can also directly access them from the menu.

    ![instana-menu-analytics](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu-analytics.png)

32. By default, we're taken to a built-in dashboard for analytics related to Application calls. We can filter by using the left side options, or by creating our own filters at the top.

33. Expand the left side options and select Only Erroneous and the Robot Shop Z application.

    ![instana-analytics-1](https://raw.githubusercontent.com/mmondics/media/main/images/instana-analytics-1.png)

    The source of some of our payment errors is starting to become apparent.

34. Click the Events option in the left side menu. Click on Incidents if you aren't automatically taken there.

    ![instana-menu-events](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu-events.png)

    ![instana-analytics-2-2](https://raw.githubusercontent.com/mmondics/media/main/images/instana-analytics-2.png)

    Instana can parse all of the requests, calls, traces, and other information it knows about into a stream of events and then classify and group them. Instana includes built-in events, predefined health signatures based on integrated algorithms which help you to understand the health of your monitored system in real-time. If a built-in event is not relevant for the monitored system, it can be disabled. Conversely, you can create a custom event in Instana if it does not already exist. These events can then be sent as an alert to a channel of your choice, such as email, Slack, Watson AIOps, Splunk, PagerDuty, Prometheus a generic webhook, or one of many more supported technologies.

    Right now we're looking at Incidents that Instana has identified during our timeframe. Incidents are created when a key performance indicator such as load, latency, or error rate changes over a certain threshold.

35. **Navigate back to the Events page and select the Issues tab.**

    ![instana-events-1](https://raw.githubusercontent.com/mmondics/media/main/images/instana-events-1.png)

    An issue is an event that is triggered if something out of the ordinary happens. You can think of Incidents as Issues that Instana has correlated with each other to form a cohesive event. Let's take a look at a single Issue.

36. **Navigate back to the Events page and select the Changes tab.**

    ![instana-events-3](https://raw.githubusercontent.com/mmondics/media/main/images/instana-events-3.png)

    A Change is an Event where Instana recognizes a change in configuration or status of a component it is monitoring, such as a deployment, pod, node, or server. Changes can be correlated with Issues under the umbrella of an Incident.

    This is the end of the navigation portion of this demonstration, so next we will start to put this knowledge to use. We just looked through a small portion of what Instana offers, but you now have a general sense of the capabilities Instana offers and how each is tied to one another through AI-driven algorithms and rules.

### Using Instana to Identify an Issue

    As you looked through the various sections of the Instana dashboard, a few errors kept popping up. In this section, we will use Instana to pinpoint the root cause of the errors and fix them. When debugging with Instana, a good place to start is Events.

37. Click the Events option in the left side menu. Click on Incidents if you aren't automatically taken there.

    ![instana-menu-events](https://raw.githubusercontent.com/mmondics/media/main/images/instana-menu-events.png)

    Instana has identified Incidents for us to look into.

38. **Click one of the Incidents titled "Erroneous call rate is too high".**

    ![instana-analytics-2](https://raw.githubusercontent.com/mmondics/media/main/images/instana-analytics-2.png)

    The incident page shows a dynamic graph of all associated issues, events, and correlates it with other incidents to provide a comprehensive overview of the situation regarding service and event impact.

39. **Click to expand the triggering event.**

    ![instana-analytics-3](https://raw.githubusercontent.com/mmondics/media/main/images/instana-analytics-3.png)

    Instana automatically displays a relevant dynamic graph. In our case, this is the erroneous call rate for the payment service. Instana also provides a link to the analysis page for these calls.

40. **Click the "Analyze Calls" button.**

    Now we're back on the Analysis page we looked at previously, but with the correct filters automatically applied. We can see that the POST /pay/{id} endpoint has 100% erroneous calls. Click to expand that dropdown.

    ![instana-analytics-4](https://raw.githubusercontent.com/mmondics/media/main/images/instana-analytics-4.png)

    Notice how we see the same information about the erroneous calls for POST /pay/partner-57 as we did previously, but Instana did all of the thinking and filtering for us. Because of the 100% error rate, it's clear that this endpoint is having an issue. Instana also provided us with links to the specific calls that failed.

41. **Click one of the "POST /pay/partner-57" links.**

    ![instana-analytics-5](https://raw.githubusercontent.com/mmondics/media/main/images/instana-analytics-5.png)

    On the call page, we see how many instances of the erroneous call there are, a trace of the call and at which point the error occurred, the status code and error messages received, and more. From reading through the information on this page, we learn that the source of the error is the payment service in Kubernetes. The related endpoints and infrastructure such as the external MongoDB and the user service look healthy.

42. **Click the "payment" link under "Service Endpoint List".**

    ![instana-analytics-6](https://raw.githubusercontent.com/mmondics/media/main/images/instana-analytics-6.png)

    And again, we confirm that the payment service in OpenShift is the root cause of these Incidents. At this point we would want to look at our Kubernetes YAML definitions and the python code that was containerized and is running this microservice. For the sake of this demonstration, we know that the error is caused by an intentional bug built into the load generator which is attempting to access an payment endpoint that does not exist.

### Instana Wrap-up

You should now have a better understanding of Instana observability, how to use the platform, and the OpenShift on IBM zSystems data and metrics it can observe. The deep and broad observability provided by Instana set the stage for other IBM solutions to **use that data** to make AI-driven insights around application performance and problem remediation.

## Turbonomic

### Overview of Turbonomic Application Resource Management 

### Navigating the Turbonomic Dashboard

### Turbonomic Actions

#### What are Actions?

#### What Actions are Available for OpenShift on IBM zSystems Targets?

#### Manually Executing Actions

#### Automatically Executing Actions

## IBM Cloud Pak for Watson AIOps

### Overview of IBM Cloud Pak for Watson AIOps

### Exploring the AI Manager Console

#### Home

#### AI Model Management

#### AIOps Insights

#### Automations

#### Data and Tool Connections

#### Resource Management

#### Stories and Alerts