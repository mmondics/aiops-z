# AIOps with IBM Z and LinuxONE
## IBM Cloud Pak for AIOps

### Introduce an Error into your Sample Application

In the following section, you will be using IBM Cloud Pak for AIOps to solve an issue in the NodeJS and Postgresql sample application running in the `userNN-project` OpenShift namespace. In this section, you will introduce the error so you can solve it later.

1. **In the OpenShift console under the developer perspective, navigate to your `userNN-project` namespace topology, click the Postgresql icon, then click the `postgresql-userNN` deployment hyperlink.**

    ![ocp-postgresql](ocp-postgresql.png)

2.  **Under the Environment tab, change the value for `POSTGRESQL_DATABASE` from `my_data` to `my_data-error`. Click save at the bottom of the page.**

    ![ocp-postgresql2](ocp-postgresql2.png)

    Your NodeJS pod will shortly become unready as it is no longer able to find the correct Postgresql database.

3.  **Try to access the frontend application with the small hyperlink button on the NodeJS icon.**

    ![app-broken](app-broken.png)

    You should receive an error page similar to the following:

    ![app-broken2](app-broken2.png)

    Once you see this error message, proceed to the next section.

### Overview of IBM Cloud Pak for AIOps

IBM Cloud Pak for AIOps is a platform that deploys advanced, explainable AI using your organization's data so that you can confidently **assess, diagnose, and resolve incidents** across mission-critical workloads and **proactively avoid incidents and accerlate your time to resolution**.

IBM Cloud Pak for AIOps helps you **uncover hidden insights from multiple sources of data**, such as logs, metrics, and events. It then **delivers those insights directly into the tools that your teams already use**, such as Slack or Microsoft Teams, in near real-time.

### Exploring the CP4AIOps Console

4. **Navigate to your IBM Cloud Pak for AIOps dashboard.** You can find this address in the [Environment Access page](../access.md).

    ![cp4waiops-login](cp4waiops-login.png)

5.  **In the dropdown for `Log in with`, make sure you have `OpenShift Authentication` selected, then select the `ldap-ats-wscdmz-wfwsldapcl01` option, and log in with your OpenShift credentials (i.e. `userNN`).** Do not select the `kube:admin` option.

    You may be prompted to authorize access to a service account in the `cp4aiops` project. Select **Allow selected permissions** if prompted.

    While navigating the CP4AIOps platform, you might be prompted to take a tour of certain features. Select "Maybe Later" - you can come back to these tours later if you wish by clicking the "Tours" button in the top-right of the page.

    ![cp4waiops-homepage](cp4waiops-homepage.png)

    When you first open CP4AIOps, you are taken to the homepage that displays the most important information that you have access to. Depending on your credentials, different "widgets" will appear for you to see and act on.

    The terms and concepts on this homepage may seem foreign at first, but they will become clear throughout the rest of this tutorial. A good place to start is on the AIOps Insights page where you can see an overview of the CP4AIOps benefits.

#### AIOps Insights

6. **Expand the menu by clicking the button in the top-left corner of the page, then navigate to the AIOps Insights page.**

    ![aiops-insights](aiops-insights.png)

    On this page, you see visualizations of two of the main goals of CP4AIOps - Improved Mean Time to Restore (MTTR) and Reduction of Noise.


    <details>
    <summary>Mean Time to Resolution (MTTR) (click to expand)</summary>
    
    The total time period from the start of a failure to resolution. For business-critical applications, downtime of just a few minutes can mean thousands or millions of dollars' worth of lost revenue. IBM Cloud Pak for AIOps reduces MTTR by using AI-driven insights to recommend actions and runbooks to solve the issue more quickly.
    </details>

    <details>
    <summary>Noise Reduction (click to expand)</summary>
    
    The concept of reducing the number of IT events and alerts that your operations staff must evaluate, speeding recovery time and reducing employee fatigue.

    In the image above, over 300,000 events were narrowed down to 10,000 alerts, which were further narrowed down to 431 incidents. These incidents are what IT Operations staff needs to evaluate and remediate either through manual processes, or by building automation for repeating incidents.

    </details>

    Next, you will take a look at where all of these events are coming from.

#### Integrations

1. **From the left-side menu, navigate to Define -> Integrations.**

    ![integrations](integrations.png)

    - All the data, events, and metrics you see in CP4AIOps are coming from the Instana server you explored earlier in this tutorial. 
    - Logs from select OpenShift applications are being forwarded to an [ELK server](https://www.elastic.co/elastic-stack) which is then ingested by CP4AIOps.
    - Slack is used for ChatOps to notify operations teams about incidents and ongoing remediation work.

    There are also two outbound connections - one for SSH connections, and one to an [Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible) server. These are both used to take remediation actions against target environments.

    *Note: don't worry if the SSH and Ansible connections indicate an error. This seems to be a visual bug due to your userNN permissions.*

#### Resource Management

8. **From the left-side menu, navigate to Operate -> Resource Management.**

    ![resource-management-2](resource-management-2.png)

    <!-- Similar to Turbonomic, you will see that CP4AIOps integrated the Instana Application Perspectives. -->

    You will see that CP4AIOps integrated the Instana Application Perspectives.

9.  **Click the link for the robot-shop application.**

    ![resource-management](resource-management.png)


    You now have a scoped view of just the resources associated with the Robot Shop Microservices Application - all of the Kubernetes objects such as pods, services, and routes, but also the individual application components within the containers such as `.jar` files and even the API calls made to each endpoint. 

    You can zoom in on the topology to better see the individual application components and relationships.

    ![resource-management-3](resource-management-3.png)

#### Automations

10. **From the left-side menu, navigate to Operate -> Automations**. If there are any filters applied, you can clear them by clicking the filter button and unchecking any that are applied.

    ![automations-policies](automations-policies.png)

    The automation tools - policies, runbooks, and actions - help you resolve incidents quickly by setting up and enabling an automatic response as situations arise.

    **Policies** are rules that contain condition and action sets. They can be triggered to automatically promote events to alerts, reduce noise by grouping alerts into an incident, and assign runbooks to remediate alerts.

11. **For example, find the Policy named `robot shop promote alert to incident`, and click it. In the new page that opens, click the "Specification" tab.**

    This policy looks for alerts that match the tags `Value of:alert.summary` contains `POST /pay/{id} - Erroneous call rate is too high`. An alert matching this tag will be sent from Instana when Instana determines there has been a significant increase in the rate of erroneous calls to the Robot Shop application.

    The policy also states what should happen when the policy finds a matching alert. In this case, it will promote the alert to an incident that will notify specific users responsible for fixing the issue, or potentially automatically run a runbook made up of one or more actions that have been defined in CP4AIOps.

12. **Navigate to the "Runbooks" tab on the Automations page.**

    ![automations-runbooks](automations-runbooks.png)

    **Runbooks** automate procedures thereby increasing the efficiency of IT operations processes. Runbooks are made up of one or more actions that can be taken against a target environment through either ssh or HTTP calls.

    You can also switch to the "Activities" tab to see all of the previous runbook usage.

13. **Navigate to the "Actions" tab on the Automations page.**

    **Actions** in runbooks are the collection of manual steps grouped into a single automated entity. An action improves runbook efficiency by automatically performing procedures and operations.

14. **For example, click the `Fix load deployment environment variable` action and then click the "Content" tab of the new window that pops up.**

    ![action-erroneous](action-erroneous.png)

    This action enables CP4AIOps to `ssh` to a target server and run the proper `oc` commands to solve the issue.

    There are other alternatives to `ssh` - for example, HTTP API calls or Ansible automation playbooks.

    Runbooks and actions can be associated with incidents so that whenever an incident is created that meets certain criteria, a runbook can automatically kick off problem remediation.

#### Incidents and Alerts

15. **In the left-side menu, navigate to Operate -> "Incidents".**

    ![incident-new](incidents.png)
    
    Depending on what alerts are triggered at the time you go through the tutorial, the current incidents will look different.

    Incidents are where the IT Operators and administrators should focus their attention to either manually close incidents as they are generated or build actions and runbooks in order to remediate incidents automatically as they appear. 

16. **Find the incident that begins with `userNN-project/nodejs-postgresql-example`, where `userNN` is your user number.**

    ***Please be careful to select your correct incident. There is nothing stopping you from accidentally selecting another user's incident and closing it in the coming steps.***

    ![incident-view](incident-view.png)

    The incident contains many pieces of information that can be used to more quickly remediate issues.

    - *Probable cause alerts* - CP4AIOps attempts to derive the root fault component, and the full scope of components that are affected by an incident
    - *Topology* - provides a view of the affected components so IT Operators can see the incident in context
    - *Assignees* - you can either manually assign incidents to team members to resolve, or CP4AIOps can assign people or teams automatically if a policy is configured to do so
    - *Impacted Applications* - any business applications that CP4AIOps identifies as impacted by the incident
    - *Recommended runbooks* - if CP4AIOps correlates the incident with others from the past that were resolved with certain playbooks, they will be recommended

17. There seems to be a problem with the NodeJS and Postgresql application running in your `userNN-project` OpenShift project. **Navigate to the OpenShift console in the developer perspective and try to access the frontend application with the small hyperlink button on the NodeJS icon.**

    ![app-broken](app-broken.png)

    You should receive an error page similar to the following:

    ![app-broken2](app-broken2.png)

    Cloud Pak for AIOps has identified this error as an *incident*, and has provided a runbook to fix it.

18. **Back in the CP4AIOps incident at the bottom of the page, click the Run button associated with the `Fix userNN postgresql (ssh)` runbook.**

    ![runbook-new](runbook-new.png)

    This *runbook* is made up of four separate *actions*. Each action is a bash command issued through an SSH connection. It is not required that all commands be of the same type. For example, one step could be a bash command while the second could be an Ansible playbook or an API call.

    - First, the runbook will log into the OpenShift cluster from an Ubuntu server via an SSH connection.
    - Second, it will check if the `POSTGRESQL_DATABASE` environment variable is properly set.
    - Third, it will remediate the error. The remediation for this error is to edit the Postgresql deployment's environment variable to the correct database name of `my_data`, rather than `my_data-error`.
    - Finally, it will check the environment variable again to confirm that it was properly changed.

19. **Start the runbook by entering the variables for `user`, `ocp_username`, `ocp_password`, and `ocp_project`.**

    - user: `userNN` (where `NN` is your user number, same as `ocp_username`)
    - ocp_username: `userNN` (where `NN` is your user number, same as `user`)
    - ocp_password: your OpenShift password found on the [Environment Access](../access.md) page
    - ocp_project: `userNN-project`

20. **After populating the variables, click *Start Runbook* at the bottom of the page.**

    You can now click the smaller *Run* buttons to manually run each step in the runbook.

21. **Click through each step in the runbook**, waiting for each step to show as **completed** before moving on.

    ![runbook-complete](runbook-complete.png)

    If the output of step 4 includes the return `POSTGRESQL_DATABASE=my_data`, the application issue should be fixed.

    You can give the runbook a 1-5 star rating, leave a comment, and mark that it worked. This will provide feedback to the automation engineers and AI algorithms so that runbooks can continue to improve.

22. **In the OpenShift console, navigate back to your `userNN-project` in the developer view. Access your frontend application by clicking the hyperlink button attached to the NodeJS application.**
   
    ![app-working](app-working.png)

    Because the NodeJS application can now reach the Postgresql database, the application is accessible and you can enter fruit quantities into the inventory database using the webpage.

    ![app-working2](app-working2.png)

    As you interact with the application, you will be able to see the resulting calls and traces in the `userNN-project` Instana application perspective. You should also see that 200 status calls have returned.

    ![instana-working](instana-working.png)

    ![instana-working2](instana-working2.png)

    In CP4AIOps, you can either manually mark your incident as *resolved*, or you can let CP4AIOps identify that the error has been fixed and it will close the incident automatically.

#### AI Model Management

Throughout this tutorial, you have been interacting with alerts, incidents, and policies that have been generated or influenced by AI algorithms that are running and training in CP4AIOps This section will show you the AI models that come pre-loaded with CP4AIOps and the benefits they provide.

23. **From the left-side menu, navigate to Operate ->  "AI Model Management".**

    ![ai-model-management](ai-model-management.png)

    This page allows you to train the pre-loaded AI models to hone their ability to derive insights from your incoming data connections (Instana in this tutorial). 

    For example, the *Temporal grouping* AI model groups alerts which co-occur over time. When a problem arises, there are typically multiple parts of a system or environment that are impacted. When alerts in different areas co-occur, it makes sense to look at them together and treat them as one problem to try and determine what might have happened. This is one of the ways that noise is reduced from the hundreds of thousands of events all the way down to a few hundred incidents.

24. **Click the "Temporal grouping" tile.**

    ![temporal-grouping](temporal-grouping.png)

    You can see the status and history of the AI model training as well as the applications that it is being applied to. Users with elevated credentials are able to manually kick of training to improve the AI model, as well as set up a schedule to automate training on a consistent basis.

## Wrapping Up

In this demonstration, you have seen some of the capabilities of IBM's AIOps portfolio and how it can observe and manage IBM Z applications and infrastructure.

<!-- With Instana, Turbonomic, and IBM Cloud Pak for AIOps, you can keep your applications up and running, meeting your SLAs, and when incidents do arise, you can remediate them quickly and get back to focusing on other projects. -->

With Instana and IBM Cloud Pak for AIOps, you can keep your applications up and running, meeting your SLAs, and when incidents do arise, you can remediate them quickly and get back to focusing on other projects.

We encourage you to look through the references below and reach out to this [tutorial author](mailto:matt.mondics@ibm.com) if you would like to see or learn more.

## References

- [IBM Cloud Pak for AIOps Product Page](https://www.ibm.com/products/cloud-pak-for-aiops)
- [IBM Cloud Pak for AIOps Documentation](https://www.ibm.com/docs/en/cloud-paks/cloud-pak-aiops)
- [IBM Cloud Pak for AIOps Integrations](https://www.ibm.com/docs/en/cloud-paks/cloud-pak-aiops/4.4.1?topic=integrations-integration-types)