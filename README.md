# Creating Argo CD Application Dependencies - Waves

## Introduction

Working with Argo CD and leveraging the GitOps approach is a great way to manage the configurations of your different clusters.
Since ApplicationSets, it is easy to automatically create different Argo CD Application objects that are rolled out to the environments. However, sometimes it happens that 
a service depends on other services before it can be deployed. By design, an Application is completely autonomous and does not know the status of another Application. Therefore, it will be challenging to configure Application dependencies and let Argo CD know when to start the deployment of the next service. 

For example, before OpenShift-Logging can be configured, the Loki Operator and a LokiStack object must have been created. You could now create a single Application object that
is responsible for both services. The problem is however that other services depend on Loki too, like the Netobserv Operator. Bundling this into the same Application will become confusing at some point as a single Argo CD Application gets huge. 

On the other hand, if you create a 2nd Application that also tries to install the Loki Operator Argo CD will show a warning message in the UI, since an object should be managed by one Application only. 

So how to deal with such an issue? The answer is to use **App-of-Apps** pattern, custom **resource health checks** and **Syncwaves**. 

## App-of-Apps
The [App-of-Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/) pattern was introduced to create a single Argo CD Application that manages other Argo CD Applications (One to rule them all). In other words: A single App-of-App that will manage the Applications that will further deploy Loki, Logging etc. In our example below, this will be the starting point.

## SyncWaves
With Syncwaves it is possible to define an order for Argo CD which will apply the managed manifests to the system. The lower the syncwave number, the earlier the object will be created. A classic example are Namespace and Deployment. It is a good thing to first deploy a Namespace before anything else:

- Namespace - Syncwave 0 
- Deployment - Syncwave 1 

This means that the Namespace object will be created first and only after it has been created (reported a healthy status to Argo CD) Argo CD will create the Deployment object. 

## Combining App-of-Apps and Syncwaves
In the following example, I will create an App-of-Apps that will take care of three other Applications that all have a Syncwave configured accordingly. The App-of-Apps will roll out the changes in "**Waves**". I created three waves are a demonstration:

- wave
- *wave
- *wave

The idea is to synchronize the App-of-App which will wait for the first Application before it starts with the second one and so on.
I am calling this "waves". The App-of-Apps is starting the waves after each other. 

Prerequisites
The be able for the App-of-Apps to verify the status of the managed Applications a Health Check must be configured in Argo CD. Such health check can be used to track the status of different Kubernetes objects. Only if the status "healthy" is returned for all Kubernetes manifests, Argo CD considers the whole Application as "healthy"

There are some built in health check. However, for the Application object the health check has been removed in Argo CD version 1.8. You can read more about this change at https://github.com/argoproj/argo-cd/issues/3781.
Well ... we can create a customer health check and bring this back. 

The following configuration can be added to the openshift-gitops operator
xxxx

If you do not use the Operator you can add the same configuration to the Argo CD ConfigMap "argocd-cm"

The syntax of such health checks is described at https://argo-cd.readthedocs.io/en/stable/operator-manual/health/#argocd-app
Bascially, it verifies if the status is heathy and reports this status to Argo CD.

## Demo
Let's create an example. 
As described above I would like to create a "wave" that deployes the services in the following order:

1. Loki Operator
2. OpenShift-Logging - creates the objects: BucketClaim, LokiStack, ClusterLogging
3. Netobserv - creates the objects: BucketClaim, LokiStack, FlowController

The example can be found at my Git repository: XXXX

The App-of-Apps (the controller) will manage three Applications that take care of the services. <----- direct git link>

BILD XXX

Alls we need to do is to create the App-of-Apps

YAML XXX

I did not configure it to automatically synchronize, so lets click the "Sync" button in Argo CD. 

BILD XXX

Now our wave will start.... XXXX 

Only when a single wave reported a healthy status, the next wave will be started. 
Eventuelly, all wave are finished and the whole App-of-App is healthy. 

BILD XXX

What about ApplicationSets?
Weren't ApplicationSets introduced to replice App-of-Apps pattern? Well the ApplicationSets are an evolution of the App-of-Apps patern. Unfortunately, there is one problem with ApplicationSets. Currently, it is not possible to configure Application dependencies with ApplicationSets. There is an open feature request that might deal with the in the future https://github.com/argoproj/applicationset/issues/221. 



```yaml
```


![Mona's Applications](images/mona-app.png)
