GitOps exploration
=====================
GitOps is a declarative approach to **continuous delivery** that uses Git as the single source of truth for everything (infrastructure and application)

![Image](./images/DeliveryModel.png)

Concepts & Architecture
=====================
RHACM can be used to automate the deployment of applications in target environments (kubernetes clusters) and keep them synchronized 

The main concepts are (CRD):

- **Cluster Hub** : the central cluster that manage all the deployments
- **managed cluster** : the clusters where the applications are deployed to

 
- **applications** : that define the groups of subscriptions that participate to each application to deploy
- **channels** : that define the git repositories
- **subscriptions** : that define the subset of manifests in a channel that defines the kubernetes objects to deploy
- **placementrules** : that define the target cluster selection rules

![Image](./images/application-model.jpg)

Installing RHACM
=====================
- RHACM is available as an operator in the OperatorHub

Simple use cases
=====================

repository architecture:
------------------------
![Image](./images/tree.jpg)

  - rhacm-def folder contains the descriptions of the various RHACM Custom resources
  - deployables folder contains everything needed to deploy applications

Cluster Hub namespaces
----------------------
- opencluster-management:

Pre-req to play with this workshop
----------------------------------
You need two clusters: one as the hub cluster which will hosts RHACM, the other as a managed cluster to deploy the applications
- fork and clone the current repository on your laptop
- install RHACM on the Hub Cluster : RHACM is available as an operator in the OperatorHub ; to complete the install, create a multiclusterhub instance
- import the managed cluster intot the Hub Cluster
- on the Hub Cluster, create a namespace to hold the channels definitions\
`oc new-project rhacm-channels`
- on the Hub Cluster, create a secret to access to the git repository\
`oc create secret generic git-secret --from-literal=user=xxxxxxx --from-literal=accessToken=xxxxxxxx -n rhacm-channels`


UC1: Add a link to the OCP Console (Config)
-------------------------------------------
TBD

UC2: Deploy a simple application (petclinic)
--------------------------------------------
- look at gitops-with-rhacm/rhacm-def/apps/apps-group1 folder\
This folder contains the RHAACM channels definitions for the group of applications and all the RHACM custom resources definitions for all the applications of the group

- look at gitops-with-rhacm/deployables/apps/apps-group1/app1/base\
This folder contains the definitions for a kubernetes deployment and a service (you can ignore the kustomization.yaml for the moment)

- create a namespace to host the custom resources definitions for the application
`oc new-project petclinic-lifecycle`
- create the RHACM Custom resources for app1 from files\
`cd gitops-with-rhacm/rhacm-def/apps/apps-group1; oc apply -f petclinic-channel.yaml; oc apply -f apps1`

- watch the resources creation from the RHACM console : as the placement rule is looking for a cluster with an app and an environment labels that it can't find yet, the application is not deployed\
![Image](./images/petclinic1.jpg)
- label the managed cluster with `app=petclinic` and `environment=dev`
- observe the deployment on the RHACM console and on the target cluster
![Image](./images/petclinic2.jpg)\


- try to scale the application up and observe that RHACM synchronizes the application back to the stage defined in Git

UC3: Add a specific route for each target environment (use kustomize)
---------------------------------------------------------------
In this use case, we will deploy the same application than for use case 2, but in several environments (dev and prod) with a custom route for each of them.

To be able to deploy a unique application to several different environments without to duplicate yaml files, we will use **kustomize** which can build a set of yaml files based on a specific directory structure separating what is common from what is specific.

In our use case ( gitops-with-rhacm/deployables/apps/apps-group1/app1 )
- the **base** folder describes everything common : the different manifests and a kustomization.yaml file that lists the ones to deploy
- the **dev** and **prod** folders define the specificities (the routes in our case) and a kustomization.yaml file describing the base location and the list of specifics

- create the RHACM Custom resources for app2 from files\
`cd gitops-with-rhacm/rhacm-def/apps/apps-group1/app2; oc apply -f petclinic-prod-appli.yaml ; oc apply -f petclinic-prod-subscription.yaml ; oc apply -f petclinic-prod-placement-rule.yaml`
- repeat with *prod* manifests
- change the value of the environment label for the managed cluster, using dev or prod
- observe the deployments in RHACM console and target clusters, depending of the label value
- test the routes

UC4: managing secrets (sealed secrets)
-------------------------------------
For private Git repositories, we need a secret to hold the credentials (user/accessToken). As secrets are not encrypted, it's impossible to store these secrets in a repository.

To avoid this, we can use **sealed secrets** ( https://github.com/bitnami-labs/sealed-secrets )

We will encrypt our Secret into a SealedSecret, which is safe to store - even to a public repository. The SealedSecret can be decrypted only by the controller running in the target cluster and nobody else (not even the original author) is able to obtain the original Secret from the SealedSecret.


UC5: use Towe for non kubernetes config
---------------------------------------

UC6: Deploy CP4I
---------------------------------------
`oc apply -k CP4I/`
`oc apply -k CP4I/operators`

security\
order dependent deployments\
objects manualy added and not described in app are not sync
