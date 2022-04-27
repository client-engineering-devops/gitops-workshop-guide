
# GitOps Workshop Guide for **Sterling File Gateway/B2B Integrator**

## Overview  

<!--- cSpell:ignore gitorg YAMLs -->

The GitOps concept originated from [Weaveworks](https://www.weave.works/) back in 2017 and the goal was to automate the operations of a Kubernetes (K8s) system using a model external to the system as the source of truth ([History of GitOps](https://www.weave.works/blog/the-history-of-gitops)).

There are various GitOps structure and workflows.  This is our opinionated point of view (PoV) on how `GitOps` can be used to manage the infrastructure, services and application layers of K8s based systems.  It takes into account the various personas interacting with the system and accounts for separation of duties.

In this workshop you will learn the following: 
-   Overview of IBM's GitOps structure and workflow.  During the presentation in part one of the workshop, the instructors will present the overview of IBM's GitOps structure and workflow.  The information that is covered can be found in the [IBM GitOps Deployment Production guide](https://production-gitops.dev/gitops/structure/#gitops-principles).
-   IBM's GitOps Receipe for deploying IBM Sterling File Gateway/B2B Integrator.  During part two of this workshop presentation, the instructors will cover an overview of [IBM GitOps Receipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/sfg-recipe.md) for deploying IBM Sterling File Gateway/B2B Integrator. 
-   Lab 1 - Deploy the IBM Sterling File Gateway/B2B Integrator using [IBM GitOps Recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/sfg-recipe.md)
-   Lab 2 - Validating Use case Requirements for Self-Healing, Upgrade/Rollback and automatic Pod Scaling.

## Lab Prerequsites - Client Environment Setup 
This part of the workshop is a hands-on lab that the instructors will walk you through to deploy an instances of IBM Sterling File Gateway/B2B Integrator.  You and a collegue will be assigned a RedHat OpenShift Environment and GitHub Organization in which to run the lab.  You will need to have your IBMId and Public GitHub ID that you provided to sign up for the lab available.  If you have any issues accessing the environment with you IBMId and GitHub ID, please consult with your lab instructor.

### Environment Assignment 
You should have received an e-mail from the IBM instructor with your assigned environment access based on your IBMid and GitHub Id. 

### Login and Setup the IBM Cloud Shell Environment

1. Access IBM Cloud shell and login to the [IBM Cloud Shell](https://cloud.ibm.com/shell)

2. Install and setup the prequiste CLIs 
```bash
mkdir bin
cd bin
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.17.4/kubeseal-0.17.4-linux-amd64.tar.gz
tar -xvf kubeseal-0.17.4-linux-amd64.tar.gz
chmod 755 kubeseal
export PATH=~/bin:$PATH
echo $PATH
```

3. Setup the environement variables
```bash
export GIT_ORG=#enter name for GitHub organization
echo $GIT_ORG #To validate that GIT_ORG has the correct value.
```
4. Clone your GitOps repositories from your Github Organization 
```bash
cd ~
mkdir $GIT_ORG
cd $GIT_ORG
git clone https://github.com/$GIT_ORG/multi-tenancy-gitops.git
git clone https://github.com/$GIT_ORG/multi-tenancy-gitops-infra.git
git clone https://github.com/$GIT_ORG/multi-tenancy-gitops-services.git
ls -l
```
5. Login to the RedHat Openshift Cluster
Log in to your cluster with your IBMid by browsing to the `OpenShift web console` (*see your environment assignment e-mail for the link to your ROKS Cluster URL*). From the dropdown menu in the upper right of the page, click Copy Login Command.  Paste the copied command in your IBM Cloud shell.


*Note that the shell session's [IBM Cloud Shell workspace](https://cloud.ibm.com/docs/cloud-shell?topic=cloud-shell-files#file-persistence) is deleted one hour after the shell session is closed.  If you loose the shell workspace, follow the steps above to re-setup the environment.*

---

## Lab 1 - Deploy the  IBM Sterling File Gateway/B2B Integrator using IBM's GitOps Recipe
Verify you can login and access ArgoCD.  See your environment assignment e-mail for the link to your ArgoCD instance*

Follow the [instructions in the GitOps Production Guide](https://production-gitops.dev/quickstart/quickstart-sterling-b2bi/#select-resources-to-deploy) to install IBM Sterling FIle Gateway.   

After you complete the validation step, make sure to DISABLE the database generation for the next section of the lab.

- Step 1:
    ```bash
    cd multi-tenancy-gitops-services/instances/ibm-sfg-b2bi
    ```
- Step 2:
  - Inside `values.yaml`, find & set 
  - ```bash
    dataSetup:
        enable: false
    dbCreateSchema: false
    ```
_  💡 **NOTE**  
> Push the changes & sync ArgoCD.
___


## Lab 2 - Validate the Use Cases for Self-Healing, Upgrade/Rollback and automatic Pod Scaling
The final part of the hands-on lab is to validate the three use cases (Self-Healing, Upgrade/RollBack, and Automatic Pod Scaling).

See the [use case instructions](./Scenarios.md) to complete final part of the workshnop.

---


### References
#### Accessing the Sterling File GateWay Console URL

1.  Retrieve the Sterling File Gateway console URL.

    ```bash
    oc get route -n tools ibm-sfg-b2bi-sfg-asi-internal-route-filegateway -o template --template='https://{{.spec.host}}'
    ```

2. Log in with the default credentials:  username:`fg_sysadmin` password: `password` 

