# Udacity Cloud Developer using Microsoft Azure Nanodegree Program - Project: Enhancing Applications

- [Introduction](#introduction)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Dependencies](#dependencies)
  - [Required Python Libraries](#required-python-libraries)
- [Local Environment Setup (Optional)](#local-environment-setup-optional)
  - [Install Redis](#install-redis)
  - [Create a Virtual Environment](#create-a-virtual-environment)
- [Azure Environment Setup](#azure-environment-setup)
  - [Azure VM Scale Set](#azure-vm-scale-set)
  - [Setup Azure Pipeline to Deploy to VM Scale Set](#setup-azure-pipeline-to-deploy-to-vm-scale-set)
- [Project Instructions](#project-instructions)
  - [Application Insights & Log Analytics](#application-insights--log-analytics)
    - [Create a Log Analytics workspace resource](#create-a-log-analytics-workspace-resource)
    - [Create an Application Insights resource](#create-an-application-insights-resource)
    - [Enable Application Insights monitoring for the VM Scale Set](#enable-application-insights-monitoring-for-the-vm-scale-set)
    - [Add the reference Application Insights to `main.py` and specify the instrumentation key](#add-the-reference-application-insights-to-mainpy-and-specify-the-instrumentation-key)
    - [Add custom event telemetry when 'Dogs' is clicked and when 'Cats' is clicked](#add-custom-event-telemetry-when-dogs-is-clicked-and-when-cats-is-clicked)
    - [Create a query to view the event telemetry in Log Analytics](#create-a-query-to-view-the-event-telemetry-in-log-analytics)
    - [Create a chart from query showing when 'Dogs' or 'Cats' is clicked](#create-a-chart-from-query-showing-when-dogs-or-cats-is-clicked)
  - [Monitoring Containers](#monitoring-containers)
    - [Cluster setup](#cluster-setup)
    - [Create an alert in Azure Monitor](#create-an-alert-in-azure-monitor)
    - [Create an autoscaler](#create-an-autoscaler)
    - [Cause load on the system](#cause-load-on-the-system)
  - [Autoscaling](#autoscaling)
    - [For the VM Scale Set, create an autoscaling rule based on metrics](#for-the-vm-scale-set-create-an-autoscaling-rule-based-on-metrics)
    - [Trigger the conditions for the rule, causing an autoscaling event](#trigger-the-conditions-for-the-rule-causing-an-autoscaling-event)
    - [When complete, enable manual scale](#when-complete-enable-manual-scale)
  - [Runbook](#runbook)
- [Submissions](#submissions)
- [Built With](#built-with)
  - [Software](#software)
  - [Open-source 3rd-party](#open-source-3rd-party)
- [Clean-up](#clean-up)
- [References](#references)
- [Requirements](#requirements)
- [License](#license)

## Introduction

In this project, you will apply the skills you have acquired in the Azure Performance course to collect and display performance and health data about an application. This is only half the battle; the other half is making informed decisions about the data and automating remediation tasks. You will use a combination of cloud technologies, such as Azure Kubernetes Service, VM Scale Sets, Application Insights, Azure Log Analytics, and Azure Runbooks to showcase your skills in diagnosing and rectifying application and infrastructure problems.

In this project, you'll be tasked to do the following:

- Setup Application Insights monitoring on a VMSS and implement monitoring in an application to collect telemetry data
- Setup an auto-scaling for a VMSS
- Setup an Azure Automation account and create a RunBook to automate the resolution of performance issues
- Create alerts to trigger auto-scaling on an AKS cluster and trigger a RunBook to execute

## Getting Started

1. Clone this repository
2. Ensure you have all the dependencies
3. Follow the environment setup guidelines and instructions below

### Prerequisites

1. [Create a free Azure Account](https://azure.microsoft.com/en-us/free/)
2. [Create a free Azure DevOps account](https://azure.microsoft.com/en-us/services/devops/?nav=min)
3. [Visual Studio Code](https://code.visualstudio.com/Download)
4. [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

### Dependencies

- [Python](https://www.python.org/downloads/)
- [Flask](https://flask.palletsprojects.com/en/1.1.x/installation/#installation)
- Redis:
  - [Non-Windows Download](https://redis.io/download)
  - [Windows Download](https://riptutorial.com/redis/example/29962/installing-and-running-redis-server-on-windows)

### Required Python Libraries

- `redis`
- `opencensus-ext-azure`
- `opencensus-ext-flask`
- `flask`

A `requirements.txt` has been provided if you want to first run the application in a local environment.

**NOTE**: The `app.run()` in `main.py` is set for your local environment. Use `app.run(host='0.0.0.0', threaded=True, debug=True)` when deploying to a VM Scale Set.

## Local Environment Setup (Optional)

If you want to run the application on localhost, follow the next steps; otherwise, you can skip to [Azure Environment Setup](#azure-environment-setup).

### Install Redis

1. Download and install `redis-server` for your operating system:

- [Redis Quick Start](https://redis.io/topics/quickstart)
- [Non-Windows](https://redis.io/download)
- [Windows](https://riptutorial.com/redis/example/29962/installing-and-running-redis-server-on-windows)

2. Start `redis-server`

### Create a Virtual Environment

Create a virtual environment by installing the dependencies and activating it:

```bash
pipenv install
pipenv shell
```

Run `main.py`:

```bash
cd azure-vote
python main.py
```

**NOTE**: The `app.run()` in `main.py` is set for your local environment. Use `app.run(host='0.0.0.0', threaded=True, debug=True)` when deploying to a VM Scale Set.

## Azure Environment Setup

### Azure VM Scale Set

A bash script has been provided to automate the creation of the VMSS. You should not need to modify this script.

**Note**: You'll need [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) installed before using this script.

1. Log in to Azure using `az login`.
2. Run `./setup-script.sh` in your terminal.

The script will take a few minutes to create and configure all resources. Once the script is complete, you can go to Azure portal and look for the **acdnd-c4-project** resource group. Inside is the VMSS resource. You'll use the public IP address and port 50000 to connect to the VM. It's port 50000 because the inbound NAT rule of the load balancer defaults to port 50000.

The following command will connect you to your VM. **Note**: Replace `[public-ip]` with the public-ip address of your VMSS:

```bash
ssh -p 50000 udacityadmin@[public-ip]
```

### Setup Azure Pipeline to Deploy to VM Scale Set

We'll use Azure Pipelines to deploy our application to an Azure VM Scale Set. Follow the step-by-step instructions [here](azure-pipelines-instructions.md).

## Project Instructions

### Application Insights & Log Analytics

#### Create a Log Analytics workspace resource

1. If you aren't already, log into Azure Portal.
2. In the existing Resource Group `acdnd-c4-project`, click "Add".
3. Type "Log Analytics workspace"
4. Click "Create"
5. Name the workspace `udacity-log`, click "Pricing Tier" and make sure "pay-as-you-go" is enabled, then click 'Review + create'

#### Create an Application Insights resource

1. In the existing Resource Group `acdnd-c4-project`, click "Add".
2. Type "Application Insights"
3. Click "Create"
4. Name the instance `udacity-appi`, and make sure to select the log analytics workspace created above, then click 'Review + create'

#### Enable Application Insights monitoring for the VM Scale Set

1. Go into Azure portal, and go to to your VM Scale Set
2. Click "Insights" on the left under the Monitoring heading
3. Click "Enable"
4. Note that the instances will be upgraded automatically; check "Instances" under the Settings heading for progress
5. Once that is complete, click "Insights" on the left under the Monitoring heading.
6. You will see several graphs that display various types of data for that instance; it could take up to 15 minutes to display the data

#### Add the reference Application Insights to `main.py` and specify the instrumentation key

The following imports are required:

```python
from opencensus.ext.azure.log_exporter import AzureLogHandler
from opencensus.ext.azure import metrics_exporter
from opencensus.stats import aggregation as aggregation_module
from opencensus.stats import measure as measure_module
from opencensus.stats import stats as stats_module
from opencensus.stats import view as view_module
from opencensus.tags import tag_map as tag_map_module
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.ext.azure.log_exporter import AzureEventHandler
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer
from opencensus.ext.flask.flask_middleware import FlaskMiddleware
```

Copy the instrumentation key from the Application Insights resource to `config_file.cfg`:

```python
APP_INSIGHTS_CONNECTION_STRING = 'InstrumentationKey={guid}'
```

#### Add custom event telemetry when 'Dogs' is clicked and when 'Cats' is clicked

See the implementation in `main.py`.

#### Create a query to view the event telemetry in Log Analytics

1. Search for your Application Insights resource and go to it.
2. Then, click "Logs" on the left side.
3. Double-click "traces" on the left side.
4. Click "Run" - this should return a result set with Cats and Dogs
5. Filter the query by adding `| where timestamp > ago(36h) | limit 10`. This will show up to 10 results which happened within the last day and a half
6. Try other ways of filtering to obtain only the results you want

#### Create a chart from query showing when 'Dogs' or 'Cats' is clicked

1. Click "Chart". This will make the results a chart
2. Change the graph type and the other parameters

### Monitoring Containers

#### Cluster setup

1. Run `az login` to login, then run `./create-cluster.sh` to create an AKS cluster and deploy a container to it
2. Once that is completed, go to Insights for the cluster
3. Observe the state of the clusters, note the number of nodes and number of containers

#### Create an alert in Azure Monitor

Create an alert in Azure Monitor to trigger when the number of pods increases over a certain threshold:

1. Click "Alerts" on the left under the Monitoring heading
2. Click "New Alert Rule"
3. Select condition `podCount`
4. Add action group
5. Give your alert a name and click "Create alert rule"

#### Create an autoscaler

Create an autoscaler by using the following Azure CLI command:

```bash
kubectl autoscale deployment azure-vote-front --cpu-percent=70 --min=1 --max=10
```

#### Cause load on the system

To cause load on the server, you can use a container and run it locally:

```bash
# Create the container and log you in to the container
kubectl run -i --tty load-generator --image=busybox /bin/sh
# Run the following command
while true; do wget -q -O- http://cluster-ip-here; done
```

After approximately 10 minutes, stop the load and observe the state of the cluster. Note the number of pods, it should have increased and should now be decreasing.

### Autoscaling

#### For the VM Scale Set, create an autoscaling rule based on metrics

1. Click "Scaling" on the left under the Settings heading
2. Select "Custom autoscale"
3. Add a new rule to scale out if the average CPU utilisation is above 40%
4. Set the instance limits to 2 minimum, 4 maximum and 2 default

#### Trigger the conditions for the rule, causing an autoscaling event

To cause load on the server, you can run the following command on the server:

```bash
cat /dev/urandom | gzip -9 | gzip -d | gzip -9 | gzip -d > /dev/null
```

After approximately 10 minutes, stop the load and observe the state of the VMSS. Note the number of instances, it should have increased and should now be decreasing.

#### When complete, enable manual scale

1. Click "Scaling" on the left under the Settings heading
2. Select "Manual scale"
3. Set "Instance count" to 2

### Runbook

1. Create an Azure Automation Account
2. Create a Runbook—either using a script or the UI—that will remedy a problem.
3. Create an alert which uses a runbook to remedy a problem.
4. Cause the problem to the flask app on the VM Scale Set.
5. Verify the problem is remedied via the Runbook.

## Submissions

1. The [`main.py`](azure-vote/main.py) which shows the code for the Application Insights telemetry data
2. [Screenshots for the Kubernetes cluster](submission-screenshots/kubernetes-cluster) which include:
   - The output of the Horizontal Pod Autoscaler, showing an increase in the number of pods
   - The Application Insights metrics which show the increase in the number of pods
   - The email you received from the alert when the pod count increased
3. [Screenshots for the Application Insights](submission-screenshots/application-insights) which include:
   - The metrics from the VM Scale Set instance - this will show CPU %, Available Memory %, Information about the Disk, and information about the bytes sent and received. There will be 7 graphs which display this data
   - Application Insight Events which show the results of clicking 'vote' for each 'Dogs' & 'Cats'
   - The output of the `traces` query in Azure Log Analytics
   - The chart created from the output of the `traces` query
4. [Screenshots for the Autoscaling of the VM Scale Set](submission-screenshots/autoscaling-vmss) which include:
   - The conditions for which autoscaling will be triggered (found in the 'Scaling' item in the VM Scale Set)
   - The Activity log of the VM scale set which shows that it scaled up with timestamp
   - The new instances being created
   - The metrics which show the load increasing, then decreasing once scaled up with timestamp
5. [Screenshots for the Azure Runbook](submission-screenshots/runbook) which include:
   - The alert configuration in Azure Monitor which shows the resource, condition, action group (this should include a reference to your Runbook), and alert rule details (may need 2 screenshots)
   - The email you received from the alert when the Runbook was executed
   - The summary of the alert which shows 'why did this alert fire?', timestamps, and the criterion in which it fired

## Built With

### Software

- [Python](https://www.python.org/downloads/) - Programming Language
- [VS Code](https://code.visualstudio.com/) - Integrated Development Environment
- [Azure DevOps](https://dev.azure.com) - Source control and pipeline creation tool

### Open-source 3rd-party

- [Azure Voting App](https://github.com/Azure-Samples/azure-voting-app-redis) - Container and sample python flask app
- [Redis](https://redis.io/) - In-memory database used for caching

## Clean-up

Clean up and remove all resources, or else you will incur charges:

```bash
az group delete --name acdnd-c4-project
```

## References

- [Set up Azure Monitor for your Python application](https://docs.microsoft.com/en-us/azure/azure-monitor/app/opencensus-python)
- [OpenCensus - A stats collection and distributed tracing framework](https://github.com/census-instrumentation/opencensus-python)
- [OpenCensus Flask Integration](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-flask)
- [Create a Log Analytics workspace in the Azure portal](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/quick-create-workspace)
- [Analyze your Azure infrastructure by using Azure Monitor logs](https://docs.microsoft.com/en-us/learn/modules/analyze-infrastructure-with-azure-monitor-logs)
- [Log queries in Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-query-overview)
- [Collect custom logs with Log Analytics agent in Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-custom-logs)
- [Create and share dashboards of Log Analytics data](https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/tutorial-logs-dashboards)
- [Usage analysis with Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/usage-overview)
- [Container insights overview](https://docs.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview)

## Requirements

Graded according to the [Project Rubric](https://review.udacity.com/#!/rubrics/2892/view).

## License

- **[MIT license](http://opensource.org/licenses/mit-license.php)**
- Copyright 2021 © [Thomas Weibel](https://github.com/thom).
