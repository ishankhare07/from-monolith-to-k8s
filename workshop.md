# Workshop 

During this workshop you will deploy a Cloud Native application, inspect it, change its configuration to use different services and 
play around with it to get familiar with Kubernetes and Cloud-Native tools that can help you to be more effective in your cloud journey. 

During this workshop you will be using GKE (Managed Kubernetes Engine inside Google Cloud) to deploy a complex application composed by multiple services. But none of the applications or tools used are tied in any way to Google infrastructure, meaning that you can run these steps in any other Kubernetes provider, as well as in an On-Prem Kubernetes installation. 

The main goal of the workshop is to guide you step by step to work with an application that you don't know but that will run on a real infrastructure (in contrast to run software in your own laptops). Due the time constraints, the workshop is focused on getting things up and running, but it opens the door for a lot of extensions and experimentation, that we encourage. 

# Pre Requisites
Here are some prerequisites to run this workshop: 

1) Google Cloud Account (if you are a QCon Plus attendee, we will provide you with one)
<details>
  <summary>Select the correct project and create a Kubernetes Cluster (Click to Expand)</summary>

Once you are logged in inside your Google Cloud account, you will need to create a Kubernetes Cluster with the following characteristics:
- 3 Nodes (n2-standard-4)
- Kubernetes API 1.16+

This will take some minutes, so you can move forward to the next step while the Kubernetes Cluster is being created.
  
</details>  
  
2) Camunda Cloud Account and Cluster, you need to sign for a [new account here](https://accounts.cloud.camunda.io/signup?campaign=workshop). 
<details>
  <summary>Login into your account and create a Cluster (Click to Expand)</summary>

Once you signed into your account, create a new cluster called `my-cluster`, you will use this in the second half of the workshop, but it is better to boot it up early on. 

</details>  



# Getting Started

On this section you will be setting up the following:
- Testing Cloud Shell and setting up aliases
- Installing Knative Serving and Eventing


Once the cluster is created, you will connect and iteract with it using Cloud Shell, a terminal that runs inside a Debian machine which comes with pre-installed tools like: 'kubectl' and 'helm'. Once you see your cluster ready in the cluster list, you can click the "Connect" button and then find the Run in Cloud Shell button, which will provision a new instance of Cloud Shell for you to use. 

Because you will be using the `kubectl` and `helm` commands a lot during the next couple of hours we recommend you to create the following aliases:

``` bash
alias k=kubectl
alias h=helm
```
Now instead of typing `kubectl` or `helm` you can just type `k` and `h` respectivily. 

You can now type inside Cloud Shell: 
``` bash
k get nodes
```

You should see something like this: 
``` bash
NAME                                           STATUS   ROLES    AGE   VERSION
gke-workshop-test-default-pool-90a86d57-cl4k   Ready    <none>   18m   v1.16.13-gke.401
gke-workshop-test-default-pool-90a86d57-g98v   Ready    <none>   18m   v1.16.13-gke.401
gke-workshop-test-default-pool-90a86d57-k0nx   Ready    <none>   18m   v1.16.13-gke.401
```

Next step you will install Knative Serving and Knative Eventing

- Knative Serving

``` bash
k apply --filename https://github.com/knative/serving/releases/download/v0.18.0/serving-crds.yaml
k apply --filename https://github.com/knative/serving/releases/download/v0.18.0/serving-core.yaml
k apply --filename https://github.com/knative/net-kourier/releases/download/v0.18.0/kourier.yaml
k patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress.class":"kourier.ingress.networking.knative.dev"}}'
k apply --filename https://github.com/knative/serving/releases/download/v0.18.0/serving-default-domain.yaml
```

You can check that the installation worked out correctly by checking that all the Knative Serving pods are running:
```bash
k get pods -n knative-serving
```

You should see something like this:
```
3scale-kourier-control-74459454f5-lhczq   1/1     Running     0          3m42s
activator-56cf848f9d-x46jq                1/1     Running     0          3m50s
autoscaler-67c75d8566-c6xrs               1/1     Running     0          3m49s
controller-6568f84b8b-sksxz               1/1     Running     0          3m49s
default-domain-x676x                      0/1     Completed   0          3m34s
webhook-785c5879fb-r9kqp                  1/1     Running     0          3m48s
```

- Knative Eventing

``` bash
k apply --filename https://github.com/knative/eventing/releases/download/v0.18.0/eventing-crds.yaml
k apply --filename https://github.com/knative/eventing/releases/download/v0.18.0/eventing-core.yaml
k apply --filename https://github.com/knative/eventing/releases/download/v0.18.0/in-memory-channel.yaml
k apply --filename https://github.com/knative/eventing/releases/download/v0.18.0/mt-channel-broker.yaml
k create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
 name: default
 namespace: default
EOF
```

You can check that the installation worked out correctly by checking that all the Knative Eventing pods are running:
``` bash
k get pods -n knative-eventing
```

You should see something like this: 
``` bash
NAME                                    READY   STATUS    RESTARTS   AGE
eventing-controller-6fc9c6cfc4-s4b78    1/1     Running   2          2m55s
eventing-webhook-667c8f6dc4-wnmjr       1/1     Running   4          2m53s
imc-controller-6c4f87765c-wcxm9         1/1     Running   0          2m14s
imc-dispatcher-74dcf4647f-4kdpz         1/1     Running   0          2m13s
mt-broker-controller-6d789b944d-hk2fc   1/1     Running   0          2m8s
mt-broker-filter-6bbcc67bc5-n2hpn       1/1     Running   0          2m9s
mt-broker-ingress-64987f6f4-4l9n6       1/1     Running   0          2m9s
```

Now you have everything ready to deploy your Cloud-Native applications to Kubernetes. 

# Deploying a Cloud Native Application

In this section you will be deploying a Conference Cloud-Native application composed by 4 simple services. 

Once you have the aliases, Knative installed and a Camunda Cloud account you can proceed to add a new `Helm Repository` where the Helm packages for the application are stored. 

You can do this by runnig the following command: 

``` bash
h repo add workshop http://chartmuseum-jx.35.222.17.41.nip.io
h repo update
```

Now you are ready to install the application by just running the following command:
``` bash
h install fmtok8s workshop/fmtok8s-app
```

The application [Helm Chart source code can be found here](https://github.com/salaboy/fmtok8s-app/).

You can check that the application running with the following two commands:
- Check the pods of the running services with: 
``` bash
k get pods
```
You should see that pods are being created or they are running:
``` bash
NAME                                                   READY   STATUS              RESTARTS   AGE
fmtok8s-agenda-h2kp8-deployment-54b8dcd9d-7c4mz        0/2     ContainerCreating   0          6s
fmtok8s-api-gateway-s5lr5-deployment-6447fc94f-4smj4   0/2     ContainerCreating   0          5s
fmtok8s-c4p-tgjvw-deployment-6796d99bd7-xh6cm          0/2     ContainerCreating   0          5s
fmtok8s-email-hdfvf-deployment-848b9bcc78-mnfkd        0/2     ContainerCreating   0          5s
```

- You can also check the Knative Services with: 
```
k get ksvc
```

You should see something like this:
``` bash
NAME                  URL                                                       LATESTCREATED               LATESTREADY                 READY   REASON
fmtok8s-agenda        http://fmtok8s-agenda.default.XXX.xip.io        fmtok8s-agenda-h2kp8        fmtok8s-agenda-h2kp8        True
fmtok8s-api-gateway   http://fmtok8s-api-gateway.default.XXX.xip.io   fmtok8s-api-gateway-s5lr5   fmtok8s-api-gateway-s5lr5   True
fmtok8s-c4p           http://fmtok8s-c4p.default.XXX.xip.io           fmtok8s-c4p-tgjvw           fmtok8s-c4p-tgjvw           True
fmtok8s-email         http://fmtok8s-email.default.XXX.xip.io         fmtok8s-email-hdfvf         fmtok8s-email-hdfvf         True
```

As soon all the pods are running and the services are ready you can copy and paste the `fmtok8s-api-gateway` URL into a different tab in your browser to access the application `http://fmtok8s-api-gateway.default.XXX.xip.io`

<img src="workshop-imgs/agenda-screen.png" alt="Conference Agenda" width="500px">

Now you can go ahead and:
1) Submit a proposal by clicking the Submit Proposal button in the main page

2) Go to the back office (top right link) and Approve or Reject the proposal

<img src="workshop-imgs/backoffice-screen.png" alt="Conference BackOffice" width="500px">

3) Check the email service to see the notification email sent to the potential speaker, this can be done with 
``` bash
k get pods
```
Where you should see the Email Service pod:
``` bash
NAME                                                   READY   STATUS    RESTARTS   AGE
fmtok8s-agenda-h2kp8-deployment-54b8dcd9d-7c4mz        2/2     Running   0          30m
fmtok8s-api-gateway-s5lr5-deployment-6447fc94f-4smj4   2/2     Running   0          30m
fmtok8s-c4p-tgjvw-deployment-6796d99bd7-xh6cm          2/2     Running   0          30m
fmtok8s-email-hdfvf-deployment-xxxxxxxxxx-mnfkd        2/2     Running   0          30m <<< this one!!
```
And then you can tail the logs by running:
``` bash
k logs -f fmtok8s-email-<YOUR POD ID> user-container
```
You should see the service logs being tailed, you can exit/stop taling the logs with `CTRL+C`.

``` bash

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.3.RELEASE)
Starting EmailService v0.0.3 on fmtok8s-email-hdfvf-deployment-848b9bcc78-mnfkd with PID 1 (/opt/app.jar started by root in /opt)
No active profile set, falling back to default profiles: default
Exposing 2 endpoint(s) beneath base path '/actuator'
Netty started on port(s): 8080
Started EmailService in 9.394 seconds (JVM running for 10.967)
```

And if you **approved** the submitted proposal you should also see something like this: 
``` bash 
+-------------------------------------------------------------------+
         Email Sent to: test@gmail.com
         Email Title: Conference Committee Communication
         Email Body: Dear test,
                 We are happy to inform you that:
                         `test` -> `test`,
                 was approved for this conference.
+-------------------------------------------------------------------+
```

4) If you approved the proposal, the proposal should pop up in the Agenda (main page) of the conference. 

Let's take a deeper look on what you just did in this section. 

# Understanding your application

In the previous section you installed an application using `Helm` which provides package management for Kubernetes application. 

For this example, there is a parent chart that contains the configuration for each of the services that is required by the application. 
You can find each of the services that are going to be deployed inside the `requirements.yaml` file defined [inside the chart here](https://github.com/salaboy/fmtok8s-app/blob/master/charts/fmtok8s-app/requirements.yaml).

This can be extended to add more components if needed, like for example adding a MongoDB and MySQL charts. 

The configuration for all these services can be found in the [`value.yaml` file here](https://github.com/salaboy/fmtok8s-app/blob/master/charts/fmtok8s-app/values.yaml). This `values.yaml` file can be overriden as well as any of the settings from each specific service when installing the chart, allowing the chart to be flexible enough to be installed with different setups. 

There are a couple of configurations to highlight for this version which are:
- [Knative Deployments are enabled](https://github.com/salaboy/fmtok8s-app/blob/master/charts/fmtok8s-app/values.yaml#L6), each service Helm Chart enable us to define if we want to use a Knative Service or a Deployment + Service + Ingress type of deployment. Because we have Knative installed, and you want to leverage Knative 
- Both the [`C4P` service](https://github.com/salaboy/fmtok8s-app/blob/master/charts/fmtok8s-app/values.yaml#L16) and the [`API Gateway` service](https://github.com/salaboy/fmtok8s-app/blob/master/charts/fmtok8s-app/values.yaml#L7) need to know where the other services are to be able to send requests. 

In this first version of the application `fmtok8s-app` all the interactions between the services happen via REST calls.

You can open different tabs in Cloud Shell to inspect the logs of each service when you are using the application (submitting and approving/rejecting proposals). 


## Chalenges
This section covers some of the challenges that you might face when working with these kind of applications inside Kubernetes. This section is not needed to continue with the workshop, but it highlight the need for some other tools to be used in conjuction with the application. 
 
<details>
  <summary>To see more details about the challenges Click to Expand</summary>

Among some of the challenges that you might face are the following big topics:
- Flow buried in code: for this scenario the `C4P` service is hosting the core business logic on how to handle new proposals. If you need to explain to non-technical people how the flow goes, you will need to dig in the code to be 100% sure about what the application is doing
- Dealing with changes: 

</details> 


## Deploying Version 2


Go to the Camunda Cloud Console, create a cluster and a client. Copy the credentials Kubernetes Secret command from the client popup and paste it into the Google Cloud Console: 
``` bash
k create secret generic camunda-cloud-secret --from-literal=ZEEBE_ADDRESS=...
```

@TODO: explain that V2 of the application is emitting events using Knative, you can observe these to understand what is happening in your application.
@TODO: explain Zeebe Cloud Events Router

``` bash 
h install fmtok8s-v2 workshop/fmtok8s-app-v2
```

# Workflow Orchestration with Camunda Cloud

``` bash
h install fmtok8s-v3 workshop/fmtok8s-app-v3
```



