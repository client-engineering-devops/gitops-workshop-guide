### Need to Disable the Database Setup before the running through the use cases 2 & 3. 

In  the `multi-tenancy-gitops-server` **repo**  turn off the database generation by editing the properties overide file `values.yaml` for the IBM Sterling B2B Integrator Service.  Execute the following:

```bash
vi ~/$GIT_ORG/multi-tenancy-gitops-services/instances/ibm-sfg-b2bi-prod/values.yaml
```

```yaml
ibm-sfg-prod:
....
  dataSetup:
    enabled: true           <--- change to false
    upgrade: false
  env:
    tz: "UTC"
    license: "accept"
    upgradeCompatibilityVerified: false
  logs:
    # true if user wish to redirect the application logs to console else false. If provided value is true , then application logs will reside inside containers. No volume mapping will be used.
    enableAppLogOnConsole: false
    #setup.cfg configuration starts here. Property names must follow camelCase format.
  setupCfg:
    ....
    dbCreateSchema: true    <----change to false 
```

Now deploy the changes by committing and pushing the changes to your `multi-tenancy-gitops-services` repository:
```bash
#change to the `multi-tenancy-gitops-services` directory
cd ~/$GIT_ORG/multi-tenancy-gitops-services

# Verify the changes, and add the files that have been changed
git status
git add -u

# Finally commit and push the changes
git commit -s -am "disable the SFG database generation"
git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

Sync the changes in Argo  via the `ibm-sfg-b2bi-prod` argo application

Now verify the the Sterling File Gateway Console.  Retrieve the Sterling File Gateway console URL.
```bash
oc get route -n b2bi-prod ibm-sfg-b2bi-sfg-asi-internal-route-dashboard -o template --template='https://{{.spec.host}}'
```
and login with the default credentials:  username:`fg_sysadmin` password: `password` 

# B2Bi Capabilities - Use Case walkthrough

## 1. Self-healing

In this section of the lab, we see how RedHat OpenShift performs self healing when a pod is deleted. 

To delete the pod, login to your cluster with your IBMid by browsing to the `OpenShift web console` (*see your environment assignment e-mail for the link to your ROKS Cluster URL*).  From the administrator section menu on the left, clink on the `Workload` drop down menu and click on Pods, and at the top, select the `tools` project.  Select one of the `ibm-sfg-b2bi-sfg` pods to delete and end of the row, click the vertical dot menu and click delete. 

  ![version](images/Delete-a-pod.png "Screenshot of Deletion")

      
After the pod is deleted, the pod is reinstantiated and processing work as part of the deployed Sterling B2B Integrator cluster.

    >💡 **NOTE**     
    > While the pod is being terminated a new pod is being created.
    > If you delete on one of the pods or it crashes Openshift will automatically create a new pod to replace the problem pod
      
  ![verion](images/terminat.png "Screenshot of termination")
      
  ![verion](images/pod-up.png "Screenshot of termination")

---

## 2. Upgrade/Rollback 

In this section of the lab, we see how you can upgrade and roll back versions using the GitOps method.

Using the GitOps method we see how the upgrade process is shorten.  We are also able to roll back to previous version if there is an issue.  The GitOps method also provides for traceability as to when and who made the change in the commit record in GitHub.

To upgrade the version, first go to the IBM Sterling Console application and check the current version, which is version `6.1.0.0`. 
![verion](images/v-1.png "Screenshot of version")

Now go update the `values.yaml` file in your repo as follows:

- Step 1:
    ```bash
    cd ~/$GIT_ORG/multi-tenancy-gitops-services/instances/ibm-sfg-b2bi
    ```
- Step 2: Inside `values.yaml`, find & set the tag from `6.1.0.0` to `6.1.0.1`
    ```yaml
  ibm-sfg-prod:
    global:
      image:
        repository: cp.icr.io/cp/ibm-sfg/sfg
        tag: 6.1.0.0                           <----change to 6.1.0.1       
    ```

Now deploy the changes by committing and pushing the changes to your `multi-tenancy-gitops-services` repository:
```bash
#change to the `multi-tenancy-gitops-services` directory
cd ~/$GIT_ORG/multi-tenancy-gitops-services

# Verify the changes, and add the files that have been changed
git status
git add -u
 
# Finally commit and push the changes
git commit -m "update to version 6.1.0.1"
git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```
Sync the changes in Argo  via the `ibm-sfg-b2bi-prod` argo application

Argocd will detect these changes and create a new pod with the latest version.

  ![verion](images/pods-termination-v0.png "Screenshot of version")
        
  ![verion](images/pods-version2.png "Screenshot of version")       


**To verify the version, simply go to the Sterling app menu and click on the support button in the Sterling Console.**  
![verion](images/newerversion.png "Screenshot of version") 


---

## 3. Horizontal Pod Autoscaling

In this section of the lab, we see how Horizontal Pod Autoscaling works in the OpenShift cluster.  We will see how the Sterling B2B Integrator instaance dynamically scales based on the load on the system.  For this lab we will simulate the load on the system by modifing the deployment paramaters via the GitOps repo. Sterling B2B Integrator can scale up and down manually or automatically.

The deployments settings below affect the load, which are the number of pods and the CPU usage. 

Before we change the settings to simulate the load,  we will increse the relicca to 2 and enable autoscaling.  Follow the steps below:

  - Step 1:
    ```bash
    cd ~/$GIT_ORG/multi-tenancy-gitops-services/instances/ibm-sfg-b2bi
    ```
  - Step 2: Inside `values.yaml`, find & set the `replicaCount` and `enabled` fields for both the `asi` and `ac` Sterling componets:

    ```yaml
    ibm-sfg-prod:
      ....
      asi:
        replicaCount: 1     <--- change to 2
        ....
        autoscaling:
          enabled: false    <----change to true
          minReplicas: 2
          maxReplicas: 4
          targetCPUUtilizationPercentage: 60
    ```
    ```yaml
    ibm-sfg-prod:
      ....
      ac:
        replicaCount: 1     <--- change to 2
        ....
        autoscaling:
          enabled: false    <----change to true
          minReplicas: 2
          maxReplicas: 4
          targetCPUUtilizationPercentage: 60
    ```
      
Now deploy the changes by committing and pushing the changes to your `multi-tenancy-gitops-services` repository:
```bash
#change to the `multi-tenancy-gitops-services` directory
cd ~/$GIT_ORG/multi-tenancy-gitops-services

# Verify the changes, and add the files that have been changed
git status
git add -u
 
# Finally commit and push the changes
git commit -m "increase replicas and enable auto scaling"
git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

Sync the changes in Argo  via the `ibm-sfg-b2bi-prod` argo application

Now, to simulate a load on the system so that we trigger the auto scaling of pods, we will lower the target CPU utiliziation by modifying the `values.yaml` file in GitOps repo.    Follow the steps below:
- Step 1:
    ```bash
    cd ~/$GIT_ORG/multi-tenancy-gitops-services/instances/ibm-sfg-b2bi
    ```
  - Step 2: Inside `values.yaml`, find & set the `replicaCount` and `enabled` fields for both the `asi` and `ac` Sterling componets:

    ```yaml
    ibm-sfg-prod:
      ....
      asi:
        replicaCount: 2   
        ....
        autoscaling:
          enabled: true  
          minReplicas: 2
          maxReplicas: 4
          targetCPUUtilizationPercentage: 60    <----change to 20
    ```
    ```yaml
    ibm-sfg-prod:
      ....
      ac:
        replicaCount: 2 
        ....
        autoscaling:
          enabled: true  
          minReplicas: 2
          maxReplicas: 4
          targetCPUUtilizationPercentage: 60    <----change to 20
    ```
      
Now deploy the changes by committing and pushing the changes to your `multi-tenancy-gitops-services` repository:
```bash
#change to the `multi-tenancy-gitops-services` directory
cd ~/$GIT_ORG/multi-tenancy-gitops-services

# Verify the changes, and add the files that have been changed
git status
git add -u
 
# Finally commit and push the changes
git commit -m "lower the target CPU utlization to simulate load."
git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

Sync the changes in Argo via the `ibm-sfg-b2bi-prod` argo application

Now go to the Redhat Openshift Console and observe the number of pods for the `asi` and `ac` Sterling componets. 

  - If a pod starts using more than 20% of the allocated CPU the autoscaler is going to spin up a new pod
    
      ![verion](images/scaleup.png "Screenshot of version")
  
  - Next, go to the Openshift console and on the left go to the drop down and search under HorizontalPodAutoscaler, you will see the new `asi` and `ac` autoscalers.

---

## 4. Managed File Transfer

In this section of the lab, we see how a trading partner can send a file using managed file transfer with Sterling File Gateway (SFG, part of Sterling B2B Integrator). We will see how, using mailboxes, trading partners can easily send files to/from each other.

Before we can upload a file from a trading partner, we need to setup all the necessary configurations first. We will configure the Partners, Template, and Channel to facilitate the transfer of a test file.

  - **Step 1: login to Sterling File Gateway as admin fg_sysadmin (go to https://route-to-asi/filegateway)**

    
      ![Login to File Gateway"](images/sfg-login.png "Login to File Gateway")
___

  - **Step 2: create a new routing channel Template named Demo_PassThrough**

    - **Click on the menu item: Routes -> Templates**

      ![Menu: Routes -> Template](images/sfg-routes-templates-s1.png "Menu: Routes -> Templates")
 &ensp;

    - **Click on the Create button on the bottom left**

      ![Menu: Routes -> Templates](images/sfg-routes-templates-s2.png "Menu: Routes -> Templates")
 &ensp;

    - **Create and configure the new Template as follows (follow the prompts and instructions on Sterling File Gateway, then click on the Next button on the bottom right to proceed to the next step)**

      ![Create New Template](images/sfg-routes-demo_passthrough-s1.png "Create New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s2.png "New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s3.png "New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s3.1.png "New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s4.png "New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s4.1.png "New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s5.png "New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s5.1.png "New Template")
 &ensp;

      ![New Template](images/sfg-routes-demo_passthrough-s5.2.png "New Template")
 &ensp;
  
    - **At the end, the new routing channel Template should look as follows**

      ![New Template](images/sfg-routes-demo_passthrough.png "New Template")
  
  - **Step 3: create the trading partners that will participate in the managed file transfer transaction**

    - **First, create a new Community named Billing where the trading partner participants will be members of**

    - **Click on the menu item: Participants -> Communities**

      ![Menu: Participants -> Communities](images/sfg-participants-s1.png "Menu: Participants -> Communities")
 &ensp;

    - **Click on Add**

      ![Add New Community](images/sfg-participants-s2.png "Add New Community")
 &ensp;
    - **Create and configure the new Community as follows**

      ![New Community](images/sfg-participants-s3.png "New Community")
 &ensp;

      ![verion](images/sfg-participants-s4.png "New Community")
 &ensp;

      ![verion](images/sfg-participants-s5.png "New Community")
 &ensp;

    - **At the end, the new Community should look as follows**

      ![verion](images/sfg-participants-s7.png "New Community")
 &ensp;

    - **Now, create the participant Partners**

    - **Click on the menu item: Participants -> Partners**

      ![Menu: Participants -> Partners](images/sfg-participants-s8.png "Menu: Participants -> Partners")
 &ensp;

    - **Click on the Create button**

      ![Create New Partner](images/sfg-participants-s9.png "Create New Partner")
 &ensp;

    - **First, create and configure the internal partner Demo_BillingSystem that will be receiving the file (the Consumer with User Name: demo_billingsystem and Password: password)**

      ![New Partner](images/sfg-participants-s10.png "New Partner")
 &ensp;

      ![verion](images/sfg-participants-s11.png "New Partner")
 &ensp;

      ![verion](images/sfg-participants-s12.png "New Partner")
 &ensp;

      ![verion](images/sfg-participants-s13.png "New Partner")
 &ensp;

      ![verion](images/sfg-participants-s14.png "New Partner")
 &ensp;

      ![verion](images/sfg-participants-s15.png "New Partner")
 &ensp;

    - **At the end, the new trading partner Demo_BillingSystem should look as follows, then click on Finish to add it**

      ![verion](images/sfg-participants-s16.png "New Partner")
 &ensp;

    - **Next, create and configure the external partner Demo_DrJohnDoe that will be sending the file (the Producer with User Name: demo_drjohndoe and Password: password)**

      ![Create New Partner](images/sfg-participants-s17.png "Create New Partner")
 &ensp;

      ![New Partner](images/sfg-participants-s18.png "New Partner")
 &ensp;

      ![New Partner](images/sfg-participants-s19.png "New Partner")
 &ensp;

      ![New Partner](images/sfg-participants-s20.png "New Partner")
 &ensp;

      ![New Partner](images/sfg-participants-s21.png "New Partner")
 &ensp;

      ![New Partner](images/sfg-participants-s22.png "New Partner")
 &ensp;

    - **At the end, the new trading partner Demo_DrJohnDoe should look as follows, then click on Finish to add it**

      ![New Partner](images/sfg-participants-s23.png "New Partner")
 &ensp;

      ![New Partner](images/sfg-participants-s24.png "New Partner")
 &ensp;

  - **Step 4: create the routing the channel between the two trading partners using the channel template created earlier**

    - **Click on the menu item: Routes -> Channels and define the producer and consumer as follows**

      ![Menu: Routes -> Channels](images/sfg-channels-s1.png "Menu: Routes -> Channels")
 &ensp;

    - **Click on the Create button at the bottom right**

      ![New Channel](images/sfg-channels-s2.png "New Channel")
 &ensp;

    - **Select the Routing Channel Template we just created (Demo_PassThrough), and also the Producer (Demo_DrJohnDoe) and the Consumer (Demo_BillingSystem)**

      ![New Channel](images/sfg-channels-s3.png "New Channel")
 &ensp;

    - **The new routing channel should look as follows**

      ![New Channel](images/sfg-channels-s4.png "New Channel")
 &ensp;

  - **Step 5: login to Sterling My File Gateway as user demo_drjohndoe and upload the test file (go to https://route-to-asi/myfilegateway)**

      ![Login to My File Gateway](images/sfg-myfg-producer-s1.png "Login to My File Gateway")
 &ensp;

    - **Copy or create the [test.txt](samples/test.txt) file and upload as follows (set Mailbox Path: /)**
      
      ![Upload File](images/sfg-myfg-producer-s2.png "Upload File")
 &ensp;

    - **Wait for the file to upload and process successfully**

      ![Upload File](images/sfg-myfg-producer-s3.png "Upload File")
 &ensp;

      ![Upload File](images/sfg-myfg-producer-s4.png "Upload File")
 &ensp;

  - **Step 6: as an Admin, check if the file routed successfully from producer (Demo_DrJohnDoe) to consumer (Demo_BillingSystem), by logging in to Sterling File Gateway as fg_sysadmin (go to https://route-to-asi/filegateway)**

      ![Login to File Gateway"](images/sfg-login.png "Login to File Gateway")
 &ensp;

    - **Click on the Find button on the top right hand side to search for the test.txt file transfer**

      ![Activity Find](images/sfg-file-admin-s1.png "Activity Find")
 &ensp;

    - **You should see the test.txt file transfer completed successfully**

      ![Activity Find](images/sfg-file-admin-s2.png "Activity Find")
 &ensp;

    - **You can click on the completed test.txt file transfer to get details of the Arrived File Events**

      ![Activity Find](images/sfg-file-admin-s3.png "Activity Find")
 &ensp;

      ![Activity Find](images/sfg-file-admin-s4.png "Activity Find")
 &ensp;

      ![Activity Find](images/sfg-file-admin-s5.png "Activity Find")
 &ensp;

  - **Step 7: finally, login to Sterling My File Gateway as user demo_billingsystem and open the received test file (go to https://route-to-asi/myfilegateway)**

      ![Login to My File Gateway](images/sfg-myfg-consumer-s1.png "Login to My File Gateway")
 &ensp;

    - **Click on the Find button on the top right had side**

      ![File Activity](images/sfg-myfg-consumer-s3.png "File Activity")
 &ensp;

    - **You should see a new Arrived File test.txt routed successfully**

      ![File Activity](images/sfg-myfg-consumer-s2.png "File Activity")
 &ensp;

    - **Click on the Route tab to get details of the Route Events**

      ![File Activity](images/sfg-myfg-consumer-s4.png "File Activity")
 &ensp;

    - **Click on the Delivery tab to get details of the Delivery Events**

      ![File Activity](images/sfg-myfg-consumer-s5.png "File Activity")
 &ensp;

    - **Then, click on the Download Files tab at the top to download and see the test.txt file**

      ![File Activity](images/sfg-myfg-consumer-s7.png "File Activity")
 &ensp;

      ![File Activity](images/sfg-myfg-consumer-s8.png "File Activity")
 &ensp;
