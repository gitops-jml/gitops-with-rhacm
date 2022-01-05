GitOps exploration
=====================
GitOps is a declarative approach to **continuous delivery** that uses Git as the single source of truth for everything (infrastructure and application)

![Image](./images/DeliveryModel.png)

Concepts & Architecture
=====================
RHACM can be used to automates the deployment of applications in target environments (kubernetes clusters) and keep them synchronized 

The main concepts are (CRD):

- **Cluster Hub** : the central cluster that manage all the deployments
- **managed cluster** : the clusters where the applications are deployed to

- **applications** : that define the groups of subscriptions that participate to each application to deploy
- **channels** : that define the git repositories
- **subscriptions** : that define the subset of manifests in a channel that defines the kubernetes objects to deploy
- **placementrules** : that define the target cluster selection rules

Installing RHACM
=====================
- RHACM is available as an operator in the OperatorHub

Simple use cases
=====================

repository architecture:
------------------------
![Image](./images/tree.jpg)

  - argo folders contain the descriptions of the various Argo CRD (applications and projects)
  - config folders contain everything related to OCP platform configuration
  - apps folders contain everything needed to deploy applications

Pre-req to play with this workshop
----------------------------------
- fork and then clone the current repository


UC1: Add a link to the OCP Console
---------------------------
- look at [console-link.yaml](./argo-crd/config/console/console-link.yaml) that defines the sources (yaml manifests) and destination (ocp cluster)

- create a new ArcoCD application from this file\
`cd gitops; oc apply -f argo-crd/config/console/console-link.yaml`

- look at the new Application in ArgoCD console.\
It's status should be Out Of Sync, because the target resources don't exist yet and the synchronization mode is Manual

![Image](./images/ConsoleApp.jpg)

- sync the application using the Sync button and wait for the Synced status.\
Then verifiy that a new link to ARgoCd documentation is added to the OCP console

![Image](./images/ConsoleLink.jpg)


UC2: Deploy a simple application (petclinic)
---------------------------
- look at  [PetClinicArgoApp.yaml](./argo/apps-def/PetClinic/PetClinicArgoApp.yaml) that defines the sources (yaml manifests) and destination (ocp cluster)

- create a new ArcoCD application from this file\
`cd gitops; oc apply -f argo/apps-def/PetClinic/PetClinicArgoApp.yaml`

- look at the new Application in ArgoCD console.\
For this application the Sync mode is automatic so you don't have to use the Sync button

![Image](./images/petclinic-outofsync.jpg)

- wait for the application to sync and watch the resources creation from the ArgoCD console

![Image](./images/petclinic-sync.jpg)

- find the route in the new namespace and test the application

![Image](./images/petclinic.jpg)

- try to scale the application and observe that ArgoCD synchronize the application back to the stage defined in Git

UC3: Add rook-ceph storage to the cluster
---------------------------
- look at [cephApp.yaml](./argo-crd/config/ceph/cephApp.yaml):\
this file defines a Application CRD for ArgoCD, that will use the content of ./argo-crd/config/ceph/ folder (yaml manifests) to create and synchronize resources in the current OCP cluster

- create a new ArcoCD application from a yaml file\
`cd gitops; oc apply -f argo-crd/config/ceph/cephApp.yaml`

- sync the new application

- wait for the sync to terminate and observe the new resources from the console

UC4: Deploy in a several environments, avoiding yaml duplication (Kustomize)
---------------------------

The **./apps-def/singlenodejs** folder describe a kustomize architecture for a small nodejs app deployed in two different environments

![Image](./images/simplenodejs-tree.jpg)

- the base folder describes everything common
- the dev and prod folders define the specificities (a label en=dev/prod and a configmap defining an environment variable)

The **./argo-crd/apps/simplenodejs** folder describes two ArgoCD application, one for DEV and the other for PROD

- use `apply -f argo-crd/apps/simplenodejs/simplenodejsAppDEV.yaml` and `apply -f argo-crd/apps/simplenodejs/simplenodejsAppPROD.yaml` to create the ArgoCD applications

- synchronize the application using ArgoCD console

- use `oc get route -n simplenodejs-dev` and `oc get route -n simplenodejs-prod` to find the route

- use the route in your browser to validate the application.\
You shoud see:
```Hello !
You've hit simplenodeapp-87957b46b-j9md2environment: DEV
```
or
```Hello !
You've hit simplenodeapp-87957b46b-j9md2environment: PROD
```

UC5: IBM implementation for deploying Software (APIC) with ArgoCD and configuring it with Tekton
---------------------------
using a specic instance of ArgoCD with specific controls and specific health checks

[APIC Tutorial](https://production-gitops.dev/guides/cp4i/apic/overview/overview/)

Challenges
=====================
secrets managements\
security\
order dependent deployments\
objects manualy added and not described in app are not sync
