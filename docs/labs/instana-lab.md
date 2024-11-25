# AIOps with IBM Z and LinuxONE

<!-- In this tutorial, you will become familiar with three of IBM's strategic AIOps solutions - Instana, Turbonomic, and IBM Cloud Pak for AIOps - and the capabilities they have to monitor and manage IBM Z applications and infrastructure. -->

In this tutorial, you will become familiar with two of IBM's strategic AIOps solutions - Instana and IBM Cloud Pak for AIOps - and the capabilities they have to monitor and manage IBM Z applications and infrastructure.

## Environment Overview

![aiops-arch](aiops-arch-AIOps-workshop.drawio.png)

## Connecting to the Lab Environment

Connection instructions with platform URLs and credentials are listed on the [Environment Access Page](../access.md).

## Exploring the Robot Shop Sample Application

1. **Open a web browser such as Firefox.**

2. In the browser, **navigate to your OpenShift console.** 

    The OpenShift console typically begins with `https://console-openshift-console-`. You can find this on the [Environment Access Page](../access.md).

    You will now see the OpenShift console login page.

    ![openshift-console-login](openshift-console-login.png)

3. **Log in with your OpenShift credentials.**

4. **Under the developer perspective, navigate to the Topology page for the `robot-shop` project.**

    ![robot-shop-topology](robot-shop-topology.png)

    Robot Shop is a simulated online store where you can purchase robots and AI solutions. Robot Shop is made up of many different microservices that are written in different programming languages. There are 12 different microservices written in languages including NodeJS, Python, Spring Boot, and Go, along with containerized databases including MongoDB, MySQL, and Redis. Each icon in the Topology represents an OpenShift deployment for a specific microservice. Each microservice is responsible for a single function of the Robot Shop application that you will see in the following steps.

5. **Open the Robot Shop web application by clicking the small button in the top right of the `web` icon.**

    This is simply a hyperlink that will take you to the Robot Shop application homepage.

    ![web-route](web-route.png)

    The Robot Shop homepage should appear like the screenshot below, with all of the same options in the left side menu.

    ![robot-shop-1](robot-shop-1.png)

6. **Explore the website and its functionalities from the left side menu.**
   
    You can register a new user, explore the catalog of purchasable robots, give them ratings, and simulate a purchase.

    Notice that from the OpenShift and Robot Shop perspectives, you don't get much of a sense of how the various microservice applications are plumbed together, how they are performing, if they have the correct amount of resources, or if any issues are affecting the application currently. In other words, there is a lack of *observability*, *application performance management*, and *proactive problem remediation*.

## Instana

### Overview of Instana Observability

Instana is an enterprise observability solution that offers application performance management - no matter where the application or infrastructure resides. Instana can monitor both containerized and traditional applications, various infrastructure types including OpenShift, public clouds & other containerization platforms, native Linux, z/OS, websites, databases, and more. The current list of supported technologies can be found [in the Instana documentation](https://www.ibm.com/docs/en/instana-observability/current?topic=configuring-monitoring-supported-technologies).

### Viewing the Instana Agent on OpenShift

7. In the OpenShift cluster, **navigate to the `instana-agent` project then click the circular icon on the Topology page that is labeled `instana-agent`.**

    ![instana-agent-daemonset](instana-agent-daemonset.png)

    This is the Instana agent that is collecting all the information about the containerized applications running on OpenShift and sending that information to the Instana server.
    
    The agent is deployed as a [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), which is a Kubernetes object that ensures one copy of the pod runs on each compute node in the cluster. Each individual Instana agent pod is responsible for the data and metrics collection for the application running on one of the compute nodes.

8. **Click the other circular icon on the Topology page that is labeled `k8sensor`.**

    ![k8sensor-deployment](k8sensor-deployment.png)

    The `k8sensor` pods are responsible for gathering information about the OpenShift cluster itself and all of the Kubernetes objects it includes - pods, namespaces, routes, etc., and sending that information to the Instana server. 

9.  **Click the `view logs` hyperlink next to one of the `k8sensor` pods in the right-side menu.**

    ```text
    2024/03/19 16:45:53 main.go:365: pod=instana-agent/k8sensor-5b46b459dc-lrz9z shards=[01 04 07 0A 0D 10 13 16]
    2024/03/19 16:45:53 main.go:365: call=senseLoop PodsCount={285} PodsRunning={282} PodsPending={3} snitch=pod sense.min=0 sense.99.9PCTL=0 sense.max=0 apply.min=2 apply.99.9PCTL=22 apply.max=27 http.do.min=0 http.do.99.9PCTL=97 http.do.max=104 encode.pmin=2.41KB encode.p99.9PCTL=34.34KB encode.pmax=254.29KB encode.tmin=0 encode.t99.9PCTL=45 encode.tmax=97 total.min=5 total.99.9PCTL=110 total.max=132 send.calls=9600 send.errors=0
    ```

    If you would like to see what data the agents are collecting or if you need to debug issues with collecting data from certain workloads running on OpenShift, these pod logs are a good first place to look.

    In the next section, you will take a look at the other side of the OpenShift Instana agent - the Instana server that is receiving the data.

### Navigating the Instana Platform

10. **In a web browser, navigate to your Instana server.** You can find this address on the [Enviroment Access Page](../access.md).

11. **Log in with your Instana credentials.**

    When you first log into Instana, you will be taken to the Home Page. This is a customizable summary page that shows the key metrics for any component of your environment in the timeframe specified in the top right of the screen.

    ![instana-timeframe](instana-timeframe.png)

12. **Set the time frame to Last Hour and click the Live button.**

    You are now seeing metrics for the environment over the previous hour, and it is updating in real time with up to 1 second granularity. Instana captures 100% of application calls and traces with no sampling, ensuring you never miss any critical data or insights into your application's performance.

    The next thing to notice is the menu on the left side of the page. If you hover over the left side panel, you will see a menu of links that will let you dive into different sections of the Instana dashboard, rather than seeing every option.

13. **Click the left side panel so the menu appears.**

    ![instana-menu](instana-menu.png)

    Next, you will go through each section in the menu.

14. **Click the Websites & Mobile Apps option in the left-side menu.**

    You can see that Instana is monitoring a website named `Robot Shop Website`. This is the set of webpages associated with the Robot Shop sample application that is deployed on OpenShift on IBM Z. Instana supports [website monitoring](https://www.ibm.com/docs/en/instana-observability/current?topic=instana-monitoring-websites) by analyzing browser request times and route loading times. It allows detailed insights into the web browsing experience of users, and deep visibility into application call paths. The Instana website monitoring solution works by using a lightweight JavaScript agent, which is embedded into the monitored website.

15. **Click the Robot Shop Website hyperlink.**

    ![instana-website-metrics](instana-website-metrics.png)

    With website monitoring, there are numerous filters and tabs with more information.

16. **You can Navigate through the various tabs to show more data - Speed, Resources, HTTP requests, and Pages.**

    In the next section, you'll explore the application metrics for the Robot Shop resources running on OpenShift.

17. **Click the Applications option in the left side menu.**

    This Instana instance has many applications configured. We will be exploring a few of them during this lab, primarily the *robot-shop* application.

    You may find it easier to use the search bar to the top right (under the timeframe selection).

18. **Click the robot-shop application hyperlink.**

    An application perspective represents a set of services and endpoints that are defined by a shared context and is declared using tags. For example, in this tutorial, the robot-shop application perspective encompasses all services and endpoints that meet the tag `kubernetes-namespace=robot-shop`.

    Alongside the Robot Shop application running in OpenShift, there is a container running a Python application that generates load to each microservice. The metrics you see now in the application perspective are coming from that load generator. At the top of the page, you can see the total number of calls, the number of erroneous calls, and the mean latency for each call over the past hour.

    As with the Websites & Mobile Applications section, the Application perspective has various tabs that contain different information. When it comes to microservices, one of the most helpful tabs in the Application perspective is the Dependency graph.

19. **Click the Dependencies tab.**

    ![instana-dependencies](instana-dependencies.png)

    The dependency graph offers a visualization of each service in the Application perspective, which services interact with each other, and visual representations of errors, high latency, or erroneous calls.

    ![robot-shop-dependencies](robot-shop-dependencies.png)

    There are more tabs in the Application perspective related to each individual service, error and log messages, the infrastructure stack related to the tag, and options to configure the Application perspective.

20. **Click through each of the tabs.**

    If you pay attention while clicking through these tabs, you will notice that the payments service has an unusually high number of erroneous calls, and you could dig into the specific calls to start debugging these errors.

    Next, you'll take a look at Instana's Kubernetes monitoring capabilities.

21. **Click the Platforms -> Kubernetes option in the left side menu.**

22. **Click the atsocpd1 (cluster) hyperlink.**

    ![instana-openshift](instana-openshift.png)

    OpenShift clusters are monitored by simply deploying a containerized Instana agent onto the cluster. Once deployed, the agent will report detailed data about the cluster and the resources deployed on it. Instana automatically discovers and monitors clusters, CronJobs, Nodes, Namespaces, Deployments, DaemonSets, StatefulSets, Services, and Pods.

    The Summary page shows the most relevant information for the cluster as a whole. The CPU, Memory, and Pod usage information are shown. The other sections, such as "Top Nodes" and "Top Deployments" show potential hotspots which you might want to have a look at.

23. **Click the Nodes tab.**

    The Nodes tab shows all of the Kubernetes nodes in the cluster in real time.

24. **Click the Namespace tab, then search for `robot-shop` and select it.**

    ![instana-ocp-ns](instana-ocp-ns.png)

    In the `robot-shop` namespace page, you can see details for all of the Kubernetes resources that are deployed in the `robot-shop` namespace, including deployments, services, pods, and the relevant metrics for each object.

25. **Click the Platforms -> IBM Z HMC option in the left side menu.**

    ![instana-zhmc-1](instana-zhmc-1.png)

    Instana supports monitoring IBM Z and LinuxONE hardware metrics and messages via the [IBM Z Hardware Management Console](https://www.ibm.com/docs/en/instana-observability/current?topic=technologies-monitoring-z-hmc) (HMC). 

    In the Instana console (and in the screenshot above), you should see two separate HMCs. HSYSHMA is managing a single DPM-mode IBM Z machine, while WSCHMC is managing three classic/Prism-mode IBM Z machines.
  
26. **Click the hyperlink for WSCHMC**

    ![instana-zhmc-2](instana-zhmc-2.png)

    This page shows you high level information about the IBM Z or LinuxONE machines themselves, including number of partitions, adapters, IP addresses, Machine Type Models and Machine Serial numbers.

    ??? Question "Pop Quiz - Which system is a z14, which is a z15, and which is a z16?"
        - QSYS: **z14** (machine type 3906)
        - FSYS: **z15** (machine type 8561)
        - KSYS: **z16** (machine type 3931)

    You may notice that there are only a small number of partitions and adapters listed for each system. This is due to the fact that Instana only displays the objects (LPARs, adapters, channels, etc) that a specific zHMC userid has access to. If you want to add or remove visibility to certain objects in Instana, you do so by managing the zHMC userid just as system administrators already do.

27. **Click the hyperlink for KSYS**

    ![instana-zhmc-3](instana-zhmc-3.png)

    You can now see the overall system utilization as well as that of individual processors. 

28. **Navigate to the "Environmental And Power" tab**

    ![instana-zhmc-4](instana-zhmc-4.png)

    Instana provides the ability to monitor your environmental and power metrics, including Temperature and Power Consumption. Many clients migrate workloads from distributed/x86 platforms to IBM Z in order to reduce power consumption and thus their carbon footprint. With the metrics on this page, you can measure the impact that these migrations have.

29. **Navigate to the "Partition" tab and expand one the KOSP3A option.**

    ![instana-zhmc-5](instana-zhmc-5.png)

    KOSP3A and KOSP3B are the names of the Logical Partitions (LPARs) where the OpenShift on IBM Z cluster is running. With this page, you can monitor the power consumption and processor usage for specific LPARs. This is useful for organizations who would like to see which applications or environments most contribute to power consumption. 

    In the next section, you will look at infrastructure metrics. In our case, the metrics related to a Linux server with an Oracle Database installed on it.

30. **Click the Infrastructure option in the left side menu.**

    ![instana-infrastructure](instana-infrastructure.png)

    The Infrastructure map provides an overview of all monitored systems. Within each group are pillars comprised of opaque blocks. Each pillar as a whole represents one agent running on the respective system. Each block within a pillar represents the software components running on that system.

31. **In the Infrastructure search bar, enter `oracledb`.** Two pillars will be returned, representing two different Oracle Databases running on Linux on IBM Z. **Select the pillar for OracleDB @acmeair.**

    ![infrastructure-oracledb](infrastructure-oracledb.png)

    You can now see information about the Oracle on IBM Z database monitored by Instana including its version, SID, and more.

32. **Click the "Open Dashboard" button for the OracleDB.**

    ![infrastructure-oracledb2](infrastructure-oracledb2.png)

    Now you see metrics specific to Oracle that a database administrator might be interested in.

    There are many IBM Z technologies supported by Instana, including z/OS. See the list of supported technologies [here](https://www.ibm.com/docs/en/instana-observability/current?topic=configuring-monitoring-supported-technologies), [here](https://www.ibm.com/docs/en/iooz/1.2.x?topic=installing-configuring-z-apm-connect-components), and [here](https://www.ibm.com/docs/en/iooz/1.2.x?topic=integrating-omegamon)

33. **Click the Analytics option in the left side menu.**

    Instana analytics are integrated into each of the panels you've looked at so far, but you can also directly access them from the menu.

    By default, you're taken to a built-in dashboard for analytics related to Application calls. You can filter by using the left side options, or by creating filters at the top.

34. **Expand the left side options and select Only Erroneous and the robot-shop application.**

    ![instana-analytics-1](instana-analytics-1.png)

    The primary source of errors is starting to become apparent.

35. **Click the Events option in the left side menu. Click the Incidents tab if you aren't automatically taken there.**

    Instana can parse all of the requests, calls, traces, and other information into a stream of events and then classify and group them. Instana includes built-in events, predefined health signatures based on integrated algorithms which help you to understand the health of your monitored system in real-time. If a built-in event is not relevant for the monitored system, it can be disabled. Conversely, you can create a custom event in Instana if it does not already exist. These events can then be sent as an alert to a channel of your choice, such as email, Slack, AIOps, Splunk, PagerDuty, Prometheus, a generic webhook, or one of many more supported technologies.

    Right now you're looking at Incidents that Instana has identified during the timeframe. Incidents are created when a key performance indicator such as load, latency, or error rate changes over a certain threshold.

36. **Select the Issues tab.**

    ![instana-events-2](instana-events-2.png)

    An issue is an event that is triggered if something out of the ordinary happens. You can think of Incidents as Issues that Instana has correlated with each other to form a cohesive event.

### Using Instana to Identify a Problem

As you looked through the various sections of the Instana dashboard, a few errors kept popping up. In this section, you will use Instana to pinpoint the root cause of the errors and fix them. When debugging with Instana, a good place to start is Events.

37. **Back on the Incidents tab, click one of the Incidents titled "Erroneous call rate is too high".**

    ![instana-events-1](instana-events-1.png)

    The incident page shows a dynamic graph of all associated issues, events, and correlates it with other incidents to provide a comprehensive overview of the situation regarding service and event impact.

38. **Click the button to open the triggering event.**

    ![instana-events-3](instana-events-3.png)

    Instana automatically displays a relevant dynamic graph. In this case, it is the erroneous call rate for the payment service. Instana also provides a link to the analysis page for these calls.

39. **Click the "Analyze Calls" button.**

    Now you're back on the Analysis page you looked at previously, but with some filters automatically applied. You can see that the `POST /pay/{id}` endpoint has 100% erroneous calls. Click to expand that dropdown.

    ![instana-analytics-1](instana-analytics-1.png)

    Notice how the same information about the erroneous calls for `POST /pay/partner-57` is displayed as it was previously, but Instana did all of the filtering. Because of the 100% error rate, it's clear that this endpoint is having an issue. Instana also provided links to the specific calls that failed.

40. **Click one of the "POST /pay/partner-57" links.**

    ![instana-analytics-2](instana-analytics-2.png)

    On the call page, you see how many instances of the erroneous call there are, a trace of the call and at which point the error occurred, the status code and error messages received, and more. From reading through the information on this page, it's apparent that the source of the error is the `payment` service in Kubernetes. The related endpoints and infrastructure such as the MongoDB and the `user` service look healthy.

41. **Click the "payment" link under "Service Endpoint List".**

    ![instana-analytics-3](instana-analytics-3.png)

    Again you can confirm that the payment service in OpenShift is the cause of these Incidents. At this point, a site reliability engineer or application owner would want to look at the Kubernetes YAML definitions and the python code that was containerized and is running this microservice. For the sake of this demonstration, the error is caused by an intentional bug built into the load generator which is attempting to access a payment endpoint that does not exist.

### Instana Wrap-up

You should now have a better understanding of Instana observability, how to use the platform, and the IBM Z data and metrics it can observe. The observability provided by Instana set the stage for other IBM solutions to **use that data** to make AI-driven insights around application performance and problem remediation.

## References

- [Instana Product Page](https://www.ibm.com/products/instana)
- [Instana Documentation](https://www.ibm.com/docs/en/instana-observability/current)
- [Instana Supported Technologies](https://www.ibm.com/docs/en/instana-observability/current?topic=configuring-monitoring-supported-technologies)
