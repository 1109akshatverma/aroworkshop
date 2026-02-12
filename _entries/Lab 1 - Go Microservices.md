# Lab 1 - Go Microservices

### Estimated Duration: 90 Minutes

## Overview

Microservices is an architectural style where applications are composed of small, independently deployable services that communicate over a network. Each service focuses on a specific business function, allowing for scalability, flexibility, and easier maintenance. Microservices enhance development speed and resilience by enabling teams to work on individual components without affecting the entire system. Now that you have your environment provisioned and the prerequisites fulfilled, it is time to start working on the labs.

Go is a fast, simple, and efficient programming language by Google, ideal for building scalable, cloud-native applications. With features like goroutines for concurrency, strong typing, and powerful built-in tools, Go enables rapid development and deployment. In the ARO lab, it’s used to build and containerize microservices with MongoDB integration, showcasing its strengths in cloud and microservice environments.

## Lab Objectives

You will be able to complete the following tasks:

- Task 1: Application Overview (READ-ONLY)
- Task 2: Connect to the cluster
- Task 3: Create Project
- Task 4: Deploy MongoDB
- Task 5: Deploy Ratings API
- Task 6: Deploy Ratings frontend
- Task 7: Create Network Policy

## Task 1: Application Overview (READ-ONLY)

In this task, you will explore the components of the rating application on Azure Red Hat OpenShift cluster and view the experience once comleting the lab.

1. You will explore the deployment of a ratings application on Azure Red Hat OpenShift.

   ![Application diagram](../media/app-overview.png)

1. The application consists of 3 components:

   | Component                                          | Link                                                                  |
   |----------------------------------------------------|--------------------------------------------------------------------   |
   | A public facing API `rating-api`                   | [GitHub repo](https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-api)             |
   | A public facing web frontend `rating-web`          | [GitHub repo](https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-web)             |
   | A MongoDB with pre-loaded data                     | [Data](https://github.com/microsoft/rating-api/raw/master/data.tar.gz)   |

1. Once you're done, you'll have an experience similar to the below.

   ![Application](../media/app-overview-1.png)

   ![Application](../media/app-overview-2.png)

   ![Application](../media/app-overview-3.png)

## Task 2: Connect to the cluster

In this task, you will log into the cluster using the `kubeadmin` user.  

1. Run the following command on **Azure Cloud Shell** to find the password for the `kubeadmin` user. 
   - Replace $CLUSTER_NAME with **ARO<inject key="Deployment ID" enableCopy="false"/>** (your OpenShift cluster name)
   - $RESOURCE_GROUP_NAME with **openshift** (the resource group name).

   ```azurecli-interactive
   az aro list-credentials \
     --name $CLUSTER_NAME \
     --resource-group $RESOURCE_GROUP_NAME
   ```

2. The following example output shows the password will be in `kubeadminPassword`.

   ```json
   {
      "kubeadminPassword": "<generated password>",
      "kubeadminUsername": "kubeadmin"
   }
   ```

   ![](../media/new/1.png)

3. **Save** these secrets in a notepad, you are going to use them to connect to the Web Portal.

## Task 3: Create Project

In this task, you will login to the web console, retrieve the login command and token and create a project.

### Login to the web console

Each Azure Red Hat OpenShift cluster has a public hostname that hosts the OpenShift Web Console.

1. Use the command `az aro list` to list the clusters in your current Azure subscription.

   ```sh
   az aro list -o table
   ```

2. You'll see the cluster web console **URL**. Open it in a new browser tab, log in with the `kubeadmin` **(1)** user and the **Password (2)** you retrieved earlier, and click **Log in (3)**.

   ![](../media/new/2.png)

   ![](../media/new/3.png)

3. After logging in, you should be able to see the **Azure Red Hat OpenShift Web Console**.

   ![](../media/new/4.png)

### Retrieve the login command and token

> **Note** Make sure you complete the [prerequisites](#Prerequisites) to install the OpenShift CLI on the Azure Cloud Shell.

1. Once you're logged into the Web Console, click on the username **kube:admin** **(1)** on the top right corner, then click **Copy login command (2)**.

   ![Copy login command](../media/redhatl2.png)

2. On the following page, click on **Display Token**.

   ![Copy login command](../media/redhatl3.png)

3. Under **Log in with this token**, copy the ```oc login``` line.

   ![Copy login command](../media/redhatl4.png)

4. Navigate back to **Azure Cloudshell** and paste the login command. You should be able to connect to the cluster.

   ![](../media/new/5.png)

### Create a project

A project allows a community of users to organize and manage their content in isolation from other communities.

1. Run the following command in Cloud Shell to create a new project:

   ```sh
   oc new-project workshop
   ```

   ![Create new project](../media/new/6.png)

> **Resources**

> * [ARO Documentation - Getting started with the CLI](https://docs.openshift.com/aro/4/cli_reference/openshift_cli/getting-started-cli.html)
> * [ARO Documentation - Projects](https://docs.openshift.com/aro/4/applications/projects/working-with-projects.html)

## Task 4: Deploy MongoDB

In this task, you will create mongoDB from Docker hub, verify if the mongoDB pod was created successfully and retrieve mongoDB service hostname

### Create mongoDB from Docker hub

Azure Red Hat OpenShift allows you to deploy a container image from Docker hub easily and we will deploy a MongoDB database service this way. The mandatory environment variables (user, password, database name etc.) can be passed in the ``oc new-app`` command line

1. Deploy the MongoDB database:

    ```sh
    oc new-app bitnami/mongodb \
      -e MONGODB_USERNAME=ratingsuser \
      -e MONGODB_PASSWORD=ratingspassword \
      -e MONGODB_DATABASE=ratingsdb \
      -e MONGODB_ROOT_USER=root \
      -e MONGODB_ROOT_PASSWORD=ratingspassword
    ```

   ![Create new project](../media/new/7.png)

2. Now, go back to the **RedHat OpenShift** web console tab. Click on the three horizontal lines **(☰) (1)** in the top left corner, expand **Administrator (2)** and click on **Developer (3)** to switch to **Developer** mode.

   ![Developer settings](../media/new/t11.png)

   >**Note:** If you see Welcome to the Developer Perspective!. You can choose **Skip tour**.

   ![Developer settings](../media/new/a1.png)

1. From the top navigation bar, click on **Project: All Projects (1)** dropdown and select the **workshop (2)** project.

   ![Developer settings](../media/new/a2.png)

1. You should see a new deployment for **mongoDB**.

### Verify if the mongoDB pod was created successfully

1. Run the `oc get all` command to view the status of the new application and verify if the deployment of the mongoDB template was successful.

    ```sh
    oc get all
    ```

   ![oc status](../media/new/a3.png)

### Retrieve mongoDB service hostname

1. Find the mongoDB service.

    ```sh
    oc get svc mongodb
    ```

   ![oc get svc](../media/new/a4.png)

1. The service will be accessible at the following DNS name: `mongodb.workshop.svc.cluster.local` which is formed of `[service name].[project name].svc.cluster.local`. This resolves only within the cluster.

1. Copy and paste the **CLUSTER IP** of the mongodb service, you'll need this IP address to import the ratings data in the next task to configure the `rating-api`.

   ![oc get svc](../media/new/a5.png)

## Task 5: Deploy Ratings API

In this task, you will fork the application to your own GitHub repository, use the OpenShift CLI to deploy the `rating-api`, configure the required environment variables, verify that the service is running, load data into Mongodb database, retrieve `rating-api` service hostname, setup GitHub webhook and update rating-api Target Port.

The `rating-api` is a NodeJS application that connects to mongoDB to retrieve and rate items. Below are some of the details that you'll need to deploy this.

- `rating-api` on GitHub: <https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-api>
- The container exposes port 8080
- MongoDB connection is configured using an environment variable called `MONGODB_URI`

### Fork the application to your own GitHub repository

1. To be able to setup CI/CD webhooks, you'll need to fork the application into your personal GitHub repository.

1. Navigate to the following GitHub Repository.

   ```
   https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-api
   ```

1. On the GitHub repository, click on **Fork** from the top-right corner.

   ![oc get svc](../media/new/a6.png)

1. On the **Create a new fork** page, click on **Create fork**. 

   ![oc get svc](../media/new/a7.png)

>**Congratulations** on completing the Task! Now, it's time to validate it. Here are the steps:
> - Hit the Validate button for the corresponding task and enter you **GitHub Username**. If you receive a success message, you have successfully validated the lab. 
> - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
> - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com.

<validation step="a2110909-a71b-4cba-8adb-f1dfd3c6a9a4" />

### Use the OpenShift CLI to deploy the `rating-api`

> **Note** You're going to be using [source-to-image (S2I)](#source-to-image-s2i) as a build strategy.

> **Note** Replace `<your GitHub username>` with your GitHub Username and then execute the command.

```sh
oc new-app https://github.com/<your GitHub username>/mslearn-aks-workshop-ratings-api --strategy=source --name=rating-api
```

![Create rating-api using oc cli](../media/new/a8.png)

### Configure the required environment variables

1. From the left pane, click on **Builds (1)** and select the **rating-api (2)**.

   ![](../media/new/t12.png)

1. Click on **Environment (1)** tab.

1. Create the `MONGODB_URI` **(2)** environment variable. This URI should look like `mongodb://[username]:[password]@[endpoint]:27017/ratingsdb` **(3)**. You'll need to replace the `[usernaame]` and `[password]` with the ones you used when creating the database. You'll also need to replace the `[endpoint]` with the hostname acquired in the previous step

1. Hit **Save (4)** when done.

   ![Create a MONGODB_URI environment variable](../media/new/t13.png)

1. It can also be done with CLI.

   ```
   oc set env deploy/rating-api MONGODB_URI=mongodb://ratingsuser:ratingspassword@mongodb.workshop.svc.cluster.local:27017/ratingsdb
   ```

   ![Create rating-api using oc cli](../media/new/a9.png)

### Verify that the service is running

1. From the left navigation pane under Developer, click on **Topology (1)** and select the **rating-api (2)** deployment to view its pod and related resources.

   ![](../media/new/b5.png)

1. Under **Resources (1)** tab, click on the pod name **rating-api-xxxxx (2)** to view its details and logs.

   ![](../media/new/b6a.png)

1. Click on the **Logs (1)** tab, you should see a log message confirming the code can **successfully (2)** connected to the mongoDB.

   ![](../media/new/b7.png)

If you navigate to the logs of the `rating-api` deployment, you should see a log message confirming the code can successfully connect to the mongoDB.
For that, in the deployment's details screen, click on *Pods* tab, then on one of the pods

![Verify mongoDB connection](../media/rating-api-working.png)

### Load data into Mongodb database

1. Clone the `ratings-api` repo by running the following command and navigate into the specific directory.

    ```sh
    git clone https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-api
    cd mslearn-aks-workshop-ratings-api/
    ```

   ![](../media/new/b9.png)

1. Get the pods. You'll specifically use the mongodb pod name to connect to the remote shell on the pod.

    ```sh
    oc get pods
    ```

   ![](../media/new/b8.png)

1. Copy the data folder into the mongoDB pod. Replace the **$MONGODB-POD-NAME** with the actual pod name.

    ```sh
    oc cp ./data $MONGODB-POD-NAME:/tmp/
    ```

1. Connect to the remote shell on the pod. Replace the **$MONGODB-POD-NAME** with the actual pod name.

    ```sh
    oc rsh $MONGODB-POD-NAME
    ```

   ![](../media/new/b10.png)

1. Run the `mongoimport` command to import the JSON data files into the database. Replace the **$MONGODB-CLUSTER-IP** with the actual mongodb service when you ran the command `oc get svc mongodb` in the previous task and verify that all the documents have been imported successfully.

    ```sh
    mongoimport --host $MONGODB-CLUSTER-IP --username ratingsuser --password ratingspassword --db ratingsdb --collection items --type json --file /tmp/data/items.json --jsonArray
    ```

    ```sh
    mongoimport --host $MONGODB-CLUSTER-IP --username ratingsuser --password ratingspassword --db ratingsdb --collection sites --type json --file /tmp/data/sites.json --jsonArray
    ```

    ```sh
    mongoimport --host $MONGODB-CLUSTER-IP --username ratingsuser --password ratingspassword --db ratingsdb --collection ratings --type json --file /tmp/data/ratings.json --jsonArray
    ```

   ![](../media/new/b11.png)

1. `Exit` the remote shell.

    ```sh
    exit
    ```

   ![](../media/new/b12.png)

### Retrieve `rating-api` service hostname

1. Find the `rating-api` service.

    ```sh
    oc get svc rating-api
    ```

1. The service will be accessible at the following DNS name over port 8080: `rating-api.workshop.svc.cluster.local:8080` which is formed of `[service name].[project name].svc.cluster.local`. This resolves only within the cluster.

### Setup GitHub webhook

1. To trigger S2I builds when you push code into your GitHib repo, you'll need to setup the GitHub webhook.

1. Retrieve the GitHub webhook trigger secret. You'll need use this secret in the GitHub webhook URL.

    ```sh
    oc get bc/rating-api -o=jsonpath='{.spec.triggers..github.secret}'
    ```

1. You'll get back something similar to the below. Make note the secret key in the red box as you'll need it in a few steps.

   ![Rating API GitHub trigger secret](../media/new/q1.png)

1. Retrieve the GitHub webhook trigger URL from the build configuration.

    ```sh
    oc describe bc/rating-api
    ```

   ![Rating API GitHub trigger url](../media/rating-api-github-webhook-url.png)

1. Replace the `<secret>` placeholder with the secret you retrieved in **Step 3** to have a URL similar to `https://api.otyvsnz3.eastus.aroapp.io:6443/apis/build.openshift.io/v1/namespaces/workshop/buildconfigs/rating-api/webhooks/SECRETSTRING/github`. You'll use this URL to setup the webhook on your GitHub repository.

1. Navigate to your **GitHub** repository, from the top menu bar, click on **Settings (1)**, select **Webhooks (2)** from the left pane, and then click on **Add webhook (3)**.

1. Paste the **URL (4)** output (similar to above) into the **Payload URL** field.

1. Change the Content Type to **application/json (5)**.

1. And then click on **Add webhook (6)**.

   ![](../media/new/q11.png)

   ![](../media/new/q12.png)

1. You should see a message from GitHub stating that your webhook was successfully configured.

1. Now, whenever you push a change to your GitHub repository, a new build will automatically start, and upon a successful build a new deployment will start.

   > **Resources**
   > * [ARO Documentation - Triggering builds](https://docs.openshift.com/aro/4/builds/triggering-builds-build-hooks.html)

### Update rating-api Target Port

1. Run the following command to edit the `rating-api` service runinng on the namepsace/project named **workshop**.

    ```sh
    oc edit svc rating-api -n workshop
    ```

1. You'll now update the 8080 port to `3000` port.

   ![](../media/edit-port-8080.png)

1. Press **i** to edit the file, update the port number to **3000**. Once updated, press *Esc* to ensure you are in Normal mode and type **:wq** and press *Enter*. This writes (saves) the changes and quits.

   ![](../media/port-3000.png)

1. You should be able to see  `service/rating-api edited`.

   ![](../media/rating-api-edited.png)

## Task 6: Deploy Ratings frontend

In this task, you will fork the application to your own GitHub repository, modify Dockerfile in your repository, use the OpenShift CLI to deploy the `rating-web`, configure the required environment variables, expose the `rating-web` service using a Route, try the service, setup GitHub webhook and make a change to the website app and see the rolling update.

The `rating-web` is a NodeJS application that connects to the `rating-api`. Below are some of the details that you'll need to deploy this.

- `rating-web` on GitHub: <https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-web>
- The rating-web frontend Dockerfile needs modification to the image to run on ARO 4.x versions
- The container exposes port 8080
- The web app connects to the API over the internal cluster DNS, using a proxy through an environment variable named `API`

### Fork the application to your own GitHub repository

To be able to setup CI/CD webhooks, you'll need to fork the application into your personal GitHub repository.

<a class="github-button" href="https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-web/fork" data-icon="octicon-repo-forked" data-size="large" aria-label="Fork MicrosoftDocs/mslearn-aks-workshop-ratings-web on GitHub">Fork</a>

>**Congratulations** on completing the Task! Now, it's time to validate it. Here are the steps:
> - Hit the Validate button for the corresponding task and enter you **GitHub Username**. If you receive a success message, you have successfully validated the lab. 
> - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
> - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com.

<validation step="9e829e84-c475-424f-9cde-e5e8cd9a873e" />

### Modify Dockerfile in your repository

> **Note** The Dockerfile needs to be modified to use the correct node.js image to build correctly.

> **Note** The `git` commands shown below have been run in a local shell pre-authenticated against the GitHub repository

1. Clone the Git repository locally and change to repo directory.

    ```sh
    git clone https://github.com/<your GitHub username>/mslearn-aks-workshop-ratings-web.git
    cd mslearn-aks-workshop-ratings-web
    ```

2. Download updated Dockerfile and Footer.vue files.

    ```sh
    wget https://raw.githubusercontent.com/sajitsasi/rating-web/master/Dockerfile -O ./Dockerfile
    wget https://raw.githubusercontent.com/sajitsasi/rating-web/master/src/components/Footer.vue -O ./src/components/Footer.vue
    ```

   ![Clone and update files](../media/clone_and_update.png)

3. Generate a Personal Access Token (Classic)

   * In the upper-right corner of your GitHub page, click your **Profile (1)**, then click on **Settings (2)**.

     ![GitHub Settings](../media/new/q13.png)

   * In the left sidebar, scroll to the bottom and click on **Developer settings**.

     ![GitHub Settings](../media/new/q14.png)

   * In Developer Settings, expand **Personal access tokens (1)**, select **Tokens (classic) (2)**, click on **Generate new token (3)** and then select **Generate new token (classic) (4)**.

     ![GitHub PAT (Classic)](../media/new/q15.png)

   * In the **Note** field, give your token a **descriptive name (1)**, leave the default settings for **Expiration (2)** and under **Select scopes**, select **repo (3)**.

     ![GitHub PAT Generate](../media/new/q16.png)

   * Scroll down to the bottom of the page and click on **Generate token**.

     ![GitHub PAT Generate](../media/new/w11.png)

   * Once the token is generated, make sure you **copy** the token in a Notepad. You will use this token for the authentication process in the next step while running the **git push** command.

     ![GitHub PAT Generate](../media/new/w12.png)

4. Verify, stage, commit and push changes to your local repository

    ```sh
    git status
    git add .
    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"
    git commit -m "Modified Dockerfile and Footer.vue"
    git push
    ```
   > **Note**: Once you run the **git push** command, make sure you enter your **GitHub Username** and the **Personal Access Token as Password** during the authentication process.

   ![Push changes to repository](../media/git-push.png)

### Use the OpenShift CLI to deploy the `rating-web`

> **Note** You're going to be using [source-to-image (S2I)](#source-to-image-s2i) as a build strategy.

1. Execute the following command in Cloudshell and replace `<your GitHub username>` with your **GitHub Username**.

   ```sh
   oc new-app https://github.com/<your GitHub username>/mslearn-aks-workshop-ratings-web --strategy=docker --name=rating-web
   ```

   > **Note:** The build will take between **5-10 minutes**.

   ![Create rating-web using oc cli](../media/oc-newapp-ratingweb.png)

### Configure the required environment variables

Create the `API` environment variable for `rating-web` Deployment Config. The value of this variable is going to be the hostname/port of the `rating-api` service.

Instead of setting the environment variable through the Azure Red Hat OpenShift Web Console, you can set it through the OpenShift CLI.

```sh
oc set env deploy rating-web API=http://rating-api:8080

```

### Expose the `rating-web` service using a Route

1. Expose the service.

   ```sh
   oc expose svc/rating-web
   ```

1. Find out the created route hostname.

   ```sh
   oc get route rating-web
   ```

1. You should get a response similar to the below.

   ![Retrieve the created route](../media/oc-get-route.png)

Notice the fully qualified domain name (FQDN) is comprised of the application name and project name by default. The remainder of the FQDN, the subdomain, is your Azure Red Hat OpenShift cluster specific apps subdomain.

### Try the service

Open the hostname in your browser, you should see the rating app page. Use **http://** port and not **https://** port to open the hostname in your browser. Play around, submit a few votes and check the leaderboard.

![rating-web homepage](../media/rating-web-homepage.png)

### Setup GitHub webhook

1. To trigger S2I builds when you push code into your GitHib repo, you'll need to setup the GitHub webhook.

1. Retrieve the GitHub webhook trigger secret. You'll need use this secret in the GitHub webhook URL.

   ```sh
   oc get bc/rating-web -o=jsonpath='{.spec.triggers..github.secret}'
   ```

1. You'll get back something similar to the below. Make note the secret key in the red box as you'll need it in a few steps.

   ![Rating Web GitHub trigger secret](../media/rating-web-github-secret.png)

1. Retrieve the GitHub webhook trigger URL from the build configuration.

   ```sh
   oc describe bc/rating-web
   ```

   ![Rating Web GitHub trigger url](../media/rating-web-github-webhook-url.png)

1. Replace the `<secret>` placeholder with the secret you retrieved in the previous step to have a URL similar to `https://api.otyvsnz3.eastus.aroapp.io:6443/apis/build.openshift.io/v1/namespaces/workshop/buildconfigs/rating-web/webhooks/SECRETSTRING/github`. You'll use this URL to setup the webhook on your GitHub repository.

1. Navigate to your **GitHub** repository, from the top menu bar, click on **Settings (1)**, select **Webhooks (2)** from the left pane, and then click on **Add webhook (3)**.

1. Paste the **URL (4)** output (similar to above) into the **Payload URL** field.

1. Change the Content Type to **application/json (5)**.

1. And then click on **Add webhook (6)**.

   ![](../media/new/q11a.png)

   ![](../media/new/q12.png)

1. You should see a message from GitHub stating that your webhook was successfully configured.

Now, whenever you push a change to your GitHub repository, a new build will automatically start, and upon a successful build a new deployment will start.

### Make a change to the website app and see the rolling update

1. Go to the `https://github.com/<your GitHub username>/mslearn-aks-workshop-ratings-web/blob/master/src/App.vue` file in your repository on GitHub.

1. Edit the file, and change the `background-color: #999;` line to be `background-color: #0071c5`.

1. Commit the changes to the file into the `master` branch.

   ![GitHub edit app](../media/new/q17a.png)

1. Immediately, go to the **Builds** tab in the OpenShift Web Console. You'll see a new build queued up which was triggered by the push. Once this is done, it will trigger a new deployment and you should see the new website color updated.

   ![Webhook build](../media/rating-web-cicd-build.png)

   ![New rating website](../media/rating-web-newcolor.png)

> **Resources**
> * [ARO Documentation - Triggering builds](https://docs.openshift.com/aro/4/builds/triggering-builds-build-hooks.html)

## Task 7: Create Network Policy

In this task, you will switch to the Cluster Console and create a network policy.

Now that you have the application working, it is time to apply some security hardening. You'll use [network policies](https://docs.openshift.com/aro/4/networking/network_policy/about-network-policy.html) to restrict communication to the `rating-api`.

### Switch to the Cluster Console

1. Select the **Developer (1)** dropdown, switch to the **Administrator (2)** console.

   ![GitHub edit app](../media/new/w13.png)

1. Make sure you're in the **workshop (1)** project, expand **Networking (2)**, select **NetwokPolicies (3)** and click **Create Network Policy (4)**.

   ![GitHub edit app](../media/new/s11.png)

### Create network policy

1. You will create a policy that applies to any pod matching the `app=rating-api` label. The policy will allow ingress only from pods matching the `app=rating-web` label.

1. Click on the **YAML view (1)**.

1. Use the **YAML (2)** below in the editor, and make sure you're targeting the **workshop** project.

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: api-allow-from-web
     namespace: workshop
   spec:
     podSelector:
       matchLabels:
         app: rating-api
     ingress:
       - from:
           - podSelector:
               matchLabels:
                 app: rating-web
   ```

   ![Create network policy](../media/create-networkpolicy.png)

1. Then click on **Create (3)**.

   ![Create network policy](../media/new/q18.png)

## Reference Links

- [Project](https://docs.openshift.com/aro/4/applications/projects/working-with-projects.html)
- [Persistent storage](https://docs.openshift.com/aro/4/storage/understanding-persistent-storage.html)
- [Source-To-Image (S2I)](https://docs.openshift.com/aro/4/openshift_images/create-images.html)
- [Routes](https://docs.openshift.com/aro/4/networking/routes/route-configuration.html)
- [Network policy](https://docs.openshift.com/aro/4/networking/network_policy/about-network-policy.html)
- [ARO Documentation - Managing Networking with Network Policy](https://docs.openshift.com/aro/4/networking/network_policy/creating-network-policy.html)

## Review

Deployed a containerized Go-based microservices application on Azure Red Hat OpenShift (ARO), using MongoDB to manage persistent data storage.

In this lab, you completed the following tasks:

- Read through the Application Overview
- Connected to the OpenShift cluster
- Created a Project
- Deployed MongoDB
- Deployed Ratings API
- Deployed Ratings frontend
- Created Network Policy for secure communication

### You have successfully completed the lab. Click on **Next >>** to proceed with the next lab.

![](../media/new/next.png)
