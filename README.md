# Custom Resource Definition

CRDs allow you to define your own custom resources, extending the Kubernetes API to fit your specific needs. 
In this blog post, we’ll walk through a practical example of how CRDs work, using a custom controller to manage ConfigMaps based on custom resources.

## Why?

CRDs are incredibly powerful because they enable you to represent and manage application-specific resources in a Kubernetes-native way. 
Here are some reasons why you might want to use CRDs:

* Abstraction: CRDs allow you to abstract away complex logic into custom resources, making it easier to manage and interact with your applications.
* Consistency: By defining custom resources, you ensure that your applications are consistently managed across different environments.
* Automation: CRDs enable automation by allowing you to define custom controllers that can watch and reconcile the state of custom resources.

## Prerequisites

Before we begin, ensure you have the following:

* Basic understanding of Kubernetes
* Kubernetes cluster with kubectl installed
* Docker (Building Controller Image)

## Steps

### Creating the Custom Resource Definition (CRD)

`kubectl apply -f ccm_crd.yaml`

### Creating the Custom Resource Instance (CR)

`kubectl apply -f ccm_cri.yaml`

### Writing the Custom Controller

[./custom_controller.py](./custom_controller.py)

### Dockerizing the Controller

`docker build -t microtema/custom-controller:v1.0.0 .`

[./Dockerfile](./Dockerfile)

### Deploying the Controller

`kubectl apply -f controller_deployment.yaml`

### Setting up RBAC for our custom controller(Role-Based Access Control)

`kubectl apply -f clusterrole.yaml`

`kubectl apply -f clusterrolebinding.yaml`

`kubectl apply -f serviceaccount.yaml`

### Testing the Example

Ensure the controller pod is running: 

`kubectl get pods`

Check if the ConfigMap is created: 

`kubectl get configmaps`

Modify the custom resource and observe the ConfigMap changes: 

`kubectl edit customconfigmaps microtema-custom-resource-instance`

Delete the custom resource and verify that the ConfigMap is deleted: 

`kubectl delete customconfigmaps microtema-custom-resource-instance`

### Testing Results

#### After CCM Creation:

Controller Pod

`kubectl get pods`

| NAME | READY | STATUS | RESTRATS | AGE |
| ---- | ----- | ------ | -------- | --- |
| custom-controller-7ff5955ff5-f49vc | 1/1  | Running | 0 | 17s |

CustomConfigmap

`kubectl get ccm`

| NAME | DATA | AGE |
| ---- | ---- | --- |
| microtema-custom-resource-instance | 1 | 17s |

CustomConfigmap

`kubectl get configmap`

| NAME | DATA | AGE |
| ---- | ---- | --- |
| kube-root-ca.crt | 1 | 1h10s |
| microtema-custom-resource-instance | 1 | 17s |

#### After CCM Delete:

`kubectl delete ccm microtema-custom-resource-instance customconfigmap.microtema.de` 
"microtema-custom-resource-instance" deleted

`kubectl get ccm`
No resources found in default namespace.

`kubectl get configmap`

| NAME | DATA | AGE |
| ---- | ---- | --- |
| kube-root-ca.crt | 1 | 1h10s |