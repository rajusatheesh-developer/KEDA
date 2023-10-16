# KEDA Kubernetes Integration

KEDA (Kubernetes Event-Driven Autoscaling) is a powerful tool that enables you to automatically scale your Kubernetes workloads based on external events or metrics. Here are the steps to integrate KEDA with your Kubernetes cluster and configure autoscaling for your workloads:

# Main Components of KEDA

KEDA, or Kubernetes Event-Driven Autoscaling, comprises several key components that work together to provide event-driven autoscaling for container workloads within a Kubernetes cluster. These components are essential for the operation of KEDA:

## 1. Scaler

The Scaler is a fundamental component of KEDA responsible for interfacing with external event sources or metrics. Scalers support various event sources and metrics, including Azure Functions, RabbitMQ, Kafka, Prometheus, and more. Each Scaler retrieves metrics or handles events from these sources and translates them into a format that KEDA can use for autoscaling.

## 2. TriggerAuthentication

TriggerAuthentication is responsible for securely authenticating with external event sources or metrics. It manages credentials and authentication mechanisms required to access the event source. This component can handle secrets or connection details necessary for interacting with event sources.

## 3. ScaledObject

A ScaledObject is a custom resource in Kubernetes that you define to specify how and when to scale your application. It establishes a connection between a Kubernetes workload (e.g., Deployment, StatefulSet) and a Scaler, defining the scaling rules and trigger configuration.

## 4. Metrics Server

The Metrics Server is an optional component that collects metrics and data from your Kubernetes cluster. It allows KEDA to collect internal metrics from your applications running in the cluster and use them for scaling. This is especially valuable for autoscaling based on custom application metrics.

## 5. Operator

The KEDA Operator is responsible for managing the lifecycle of ScaledObjects and ensuring that your defined scaling rules are applied to the associated Kubernetes workloads. It continuously monitors the cluster for changes

For furthur, please refer : https://keda.sh/docs/2.11/

## Prerequisites

- You need a running Kubernetes cluster. You can set up a cluster using a managed service like Azure Kubernetes Service (AKS), Google Kubernetes Engine (GKE), or on your own infrastructure using tools like Minikube or kubeadm.

- Install the `kubectl` command-line tool for managing your Kubernetes cluster.

- Install Helm, the package manager for Kubernetes.

## Install KEDA

To install KEDA, you can use Helm. Here are the steps to install KEDA:

```bash
# Add the KEDA Helm repository
helm repo add kedacore https://kedacore.github.io/charts

# Create a custom namespace for KEDA (optional but recommended)
kubectl create namespace keda

# Install KEDA into the keda namespace
helm install keda kedacore/keda --namespace keda
```
# Create a Trigger

KEDA uses triggers to monitor external events or metrics and scale your workloads accordingly. You can create triggers for various event sources like Azure Functions, Kafka, Prometheus, and more. Here's an example of creating a trigger for an Azure Functions event:

```yaml
apiVersion: keda.k8s.io/v1alpha1
kind: TriggerAuthentication
metadata:
  name: azure-functions-auth
spec:
  # Add your authentication details here
  secretTargetRef:
    - parameter: azure-functions-secret
      name: my-secret
  triggerAuthentication: {}
---
apiVersion: keda.k8s.io/v1alpha1
kind: Trigger
metadata:
  name: azure-functions-trigger
spec:
  authenticationRef: azure-functions-auth
  pollingInterval: 15
  broker: azure-queue
  functionSelector:
    functionSelector: "MyFunctionName"
  metadata:
    target: azure-functions
```

# Authentication Parameters

When working with Azure Service Bus, you have two primary authentication methods: Connection String Authentication and Pod Identity-based Authentication.

## Connection String Authentication

You can authenticate using a connection string for the Azure Service Bus Namespace. Supported formats include:

- **With SharedAccessKey**:
    ```
    Endpoint=sb://<sb>.servicebus.windows.net/;SharedAccessKeyName=<key name>;SharedAccessKey=<key value>
    ```

- **With SharedAccessSignature**:
    ```
    Endpoint=sb://<sb>.servicebus.windows.net/;SharedAccessSignature=SharedAccessSignature sig=<signature-string>&se=<expiry>&skn=<keyName>&sr=<URL-encoded-resourceURI>
    ```

Refer to [this page](#) for more information on using Shared Access Signatures.

## Pod Identity-Based Authentication

For pod-level authentication, you can use Azure AD Pod Identity or Azure AD Workload Identity providers.

These authentication methods allow you to securely access Azure Service Bus resources from your Kubernetes pods using identity-based authentication.


Adjust the trigger configuration to match your event source and authentication requirements.
For furthur reading, please refer: https://keda.sh/docs/2.11/scalers/
# Deploy and Configure Autoscaling

Next, you'll deploy a Kubernetes Deployment, StatefulSet, or any other scalable workload and configure it for autoscaling. To do this, add the scaledObject specification to your workload's YAML. Here's an example of autoscaling a Kubernetes Deployment:

```yaml
apiVersion: keda.k8s.io/v1alpha1
kind: ScaledObject
metadata:
  name: sample-scaledobject
spec:
  scaleTargetRef:
    name: sample-deployment
  minReplicaCount: 1
  maxReplicaCount: 10
  pollingInterval: 5
triggers:
  - type: azure-functions-trigger
```
# Parameter List for Azure Service Bus Scaler

The following is a list of parameters for configuring the Azure Service Bus Scaler, which allows you to perform autoscaling based on the message count in your Azure Service Bus queue or topic.

- **messageCount**: The amount of active messages in your Azure Service Bus queue or topic to scale on.

- **activationMessageCount**: The target value for activating the scaler. Learn more about activation [here](#). (Default: 0, Optional)

- **queueName**: Name of the Azure Service Bus queue to scale on. (Optional)

- **topicName**: Name of the Azure Service Bus topic to scale on. (Optional)

- **subscriptionName**: Name of the Azure Service Bus queue to scale on. (Optional*, Required when topicName is specified)

- **namespace**: Name of the Azure Service Bus namespace that contains your queue or topic. (Optional*, Required when pod identity is used)

- **connectionFromEnv**: Name of the environment variable your deployment uses to get the connection string of the Azure Service Bus namespace. (Optional)

- **useRegex**: Provides an indication of whether or not a regex is used in the `queueName` or `subscriptionName` parameters. (Values: true, false, Default: false, Optional)

- **operation**: Defines how to compute the number of messages when `useRegex` is set to true. (Values: sum, max, or avg, Default: sum, Optional).

- **cloud**: Name of the cloud environment that the service bus belongs to. Must be a known Azure cloud environment, or "Private" for Azure Stack Hub or Air Gapped clouds. (Valid values: AzurePublicCloud, AzureUSGovernmentCloud, AzureChinaCloud, AzureGermanCloud, Private; default: AzurePublicCloud)

These parameters are used to configure and fine-tune the behavior of the Azure Service Bus Scaler for autoscaling your application based on the message count in Azure Service Bus.


# KEDA Scaling :
please refer : https://keda.sh/docs/2.10/concepts/scaling-deployments/
#  Monitor and Observe

Once configured, KEDA will autonomously manage the scaling of your workload. You can monitor the scaling events and metrics using KEDA's metrics and logs to ensure that it operates as expected.

This detailed guide explains how to integrate KEDA with Kubernetes to achieve event-driven autoscaling for your container workloads. The specific configuration and integration steps may vary depending on your workload and the event/metric source you are using.

For more information and advanced configurations, consult KEDA's documentation and the documentation for your chosen event source.
