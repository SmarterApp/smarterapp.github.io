---
title: Deploy to Kubernetes
permalink: "/deployment/checklist/student_3x/deploy_to_kubernetes.html"
layout: "document"
categories: ["deployment", "checklist", "tds", "3x"]
---
# Deploy to Kubernetes

**NOTE:**  This checklist assumes that a user conducting the deployment has access to an AWS account with sufficient privileges to interact with the following services:

* VPC
* EC2
* Route53
* ElastiCache
* S3

### Create a VPC
Create a new VPC that will host the Kubernetes environment:

* Identify the region and zone(s) in which the VPC should reside
* Allocate a new Elastic IP Address
* Create the new VPC using the VPC Wizard:
  * Choose the VPC configuration that has **Public and Private Subnets**
  * ***OPTIONAL:*** provide names for the public and private subnets
  * ***IMPORTANT*:**{: style="color: #f00"} <span style=" background-color: #ff0;">Do not create any additional subnets; `kops` will create the subnets it needs as the cluster is created</span>
* Record the VPC id; it will be used later when creating the cluster

***IMPORTANT*:**{: style="color: #f00"} <span style=" background-color: #ff0;">When choosing a region that will host the Kubernetes cluster, make sure there are enough Elastic IPs available for allocation.  One Elastic IP is required for each *node* that will be a member of the Kubernetes cluster.  For example, if the cluster consists of four nodes plus a bastion node, five Elastic IP addresses are required.  If there are not enough Elastic IP addresses available, `kops` will not be able to create the cluster.</span>

### Create an S3 Bucket to Store Cluster Configuration
* Create an S3 bucket for storing configuration using console or CLI.
  `aws s3 mb s3://`[*bucket name*{: style="color: #f00;"}] `--region `[*AWS region*{: style="color: #f00;"}]
* Example:

  <div class="highlighter-rouge" style="display: inline-flex;">
    <pre class="highlight">
      <code>
        aws s3 mb s3://<span class="placeholder-example">kops-tds-cluster-state-store</span> --region <span class="placeholder-example">us-east-1</span>
      </code>
    </pre>
  </div>
* Apply versioning to the bucket:

  `aws s3api put-bucket-versioning --bucket `[*bucket name*{: style="color: #f00"}] `--versioning-configuration Status=Enabled`
* Example:
  <div class="highlighter-rouge" style="display: inline-flex;">
    <pre class="highlight">
      <code>
        aws s3api put-bucket-versioning --bucket <span class="placeholder-example">kops-tds-cluster-state-store</span> --versioning-configuration Status=Enabled
      </code>
    </pre>
  </div>
* Record kops bucket for future reference.
* ***OPTIONAL:*** Add the path to the S3 bucket to the profile:  `export KOPS_STATE_STORE=s3://`[*bucket name*{: style="color: #f00;"}]

### Generate Key for Accessing the Ops System
* Generate ssh key for accessing the ops system and for cluster (later on). You can generate locally and upload to AWS Key Pairs, or generate via AWS and keep private key local, or ...

`ssh-keygen -t rsa -C "`[*brief comment that describes the purpose of this file*{: style="color: #f00;"}]`" -f `[*the name of the file*{:style="color: #f00;"}]

Example:

<div class="highlighter-rouge" style="display: inline-flex;">
  <pre class="highlight">
    <code>
      ssh-keygen -t rsa -C "<span class="placeholder-example">key for k8s dev environment</span>" -f <span class="placeholder-example">tds_ops_dev</span>
    </code>
  </pre>
</div>

### Create Kubernetes Cluster
This section covers creating and initializing the base Kubernetes cluster. Whoever is deploying the system should decide what values to use for each command and remain consistent throughout as our examples have done below.

1. Determine the name of the cluster. 
For the examples below we are using `<ZONE>`=`us-west-2a` and `<CLUSTER>`=`tdsuat.sbtds.org` because `sbtds.org` is hosted on AWS.
1. Create the cluster configuration:
  <div class="highlighter-rouge" style="display: inline-flex;">
    <pre class="highlight">
      <code>
        kops create cluster \
          --cloud=aws \
          --node-count=<span class="placeholder">[the number of nodes desired for the cluster]</span> \
          --zones=<span class="placeholder">[The AWS region where the previously created VPC resides]</span> \
          --master-zones=<span class="placeholder">[a comma-separated list of AWS zones]</span> \
          --dns-zone=<span class="placeholder">[the Route53 hosted zone]</span> \
          --vpc="<span class="placeholder">[the id of the previously created VPC]</span>" \
          --network-cidr="<span class="placeholder">[the CIDR range for the **private subnet** of the VPC]</span>" \
          --node-size=<span class="placeholder">[The instance size of each agent node in the cluster]</span> \
          --master-size=<span class="placeholder">[The instance size of the master node in the cluster]</span> \
          --ssh-public-key="<span class="placeholder">[The path to the ssh key created in the previous step]</span>" \
          --topology private --networking weave --bastion \
          --state <span class="placeholder">[the path to the S3 bucket that stores the cluster configuration]</span> \
          --name <span class="placeholder">[the name of the cluster]</span>
      </code>
    </pre>
  </div>
  * Example:
  <div class="highlighter-rouge" style="display: inline-flex;">
    <pre class="highlight">
      <code>
        kops create cluster \
          --cloud=aws \
          --node-count=<span class="placeholder-example">4</span> \
          --zones=<span class="placeholder-example">us-east-1a</span> \
          --master-zones=<span class="placeholder-example">us-east-1a</span> \
          --dns-zone=<span class="placeholder-example">sbtds.org</span> \
          --vpc="<span class="placeholder-example">vpc-2348765b</span>" \
          --network-cidr="<span class="placeholder-example">170.20.0.0/16</span>" \
          --node-size=<span class="placeholder-example">m3.large</span> \
          --master-size=<span class="placeholder-example">m3.large</span> \
          --ssh-public-key="<span class="placeholder-example">~/.ssh/tds_ops_dev.pub</span>" \
          --topology private --networking weave --bastion \
          --state <span class="placeholder-example">s3://kops-tds-cluster-state-store</span> \
          --name <span class="placeholder-example">tdsuat.sbtds.org</span>
      </code>
    </pre>
  </div>
  - Example: `kops create cluster --zones=us-west-2a tdsuat.sbtds.org`
1. ***OPTIONAL:*** Configure node instance group (e.g. to change min/max size, etc)
  - Run `kops edit ig nodes --name `[*name of cluster*{: style="color: #f00;"}] 
  - Example: `kops edit ig nodes --name tdsuat.sbtds.org`
1. Apply the configuration to create the cluster
  - Run `kops update cluster `[*name of cluster*{: style="color: #f00;"}] `--yes`
  - Example: `kops update cluster tdsuat.sbtds.org --yes`
  - Wait for cluster to initialize (this can take up to 20 minutes).  You can run the following commands to verify the cluster:
    - `kops validate cluster`
    - `kubectl get nodes --show-labels`

### Create Redis Cluster
Create a single node AWS ElastiCache Redis.

- The Redis Cache is created within the same VPC as the kubernetes nodes
- The security group allows access to the kubernetes container on 6379 (default unless changed by you)

#### Running `kubectl` Commands Against the Cluster

From here, all `kubectl` commands should be executed from a machine that has access to the VPC that was created (e.g. the bastion server that was created as part of the cluster or a "jump box" that has sufficient privileges to interact with the cluster).

If using the bastion server, `kops` and `kubectl` will likely have to be installed and configured.  After the cluster is successfully created, the output will display the command necessary to `ssh` into the bastion server.  Alternately, instructions can be found [here](https://github.com/kubernetes/kops/blob/master/docs/bastion.md).

***OPTIONAL:*** Add the kubernetes dashboard:
* Run `kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/kubernetes-dashboard/v1.6.0.yaml`

***OPTIONAL:*** Add monitoring via heapster
* Run `kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/monitoring-standalone/v1.6.0.yaml`

### Create Redis Cluster
Create a single node AWS ElastiCache Redis.

- The Redis Cache is created within the same VPC as the kubernetes nodes
- The security group allows access to the kubernetes container on 6379 (default unless changed by you)

### Create NGINX
This creates the NGINX ingress controller.

- Run `kubectl create -f nginx-ingress-init.yml`

### Create Rabbit
Create a RabbitMQ cluster that will be used by the application.  For convenience, we have included a cluster deployment within the Kubernetes environment, if one is not already available.

1. Go to the `deployment` folder located at wherever you unzipped tds_deployment_support_files.zip
1. Run `kubectl create -f rabbit-cluster.yml`
  - This will start up three rabbit containers.  You need to wait for those to stand up before moving onto the next steps.
  - All rabbit servers need to be “Running” at 1/1 via Kubectl.  You can get this information by running `kubectl get po -w`

1. Ensure the rabbit cluster started properly
  - Run `kubectl logs -f rabbit-server-0`
  - The above command will stream logs to standard out.  You should see the rabbit nodes joining the cluster.  The example logs below show the first node joining the cluster.
1. Setup username & password for Rabbit
  - Run `kubectl exec -it rabbit-server-0 /bin/bash` opening an interactive bash terminal to the pod.
  - Run `export RABBITMQ_NODENAME=rabbit@$(hostname -f)`
  - Create the user and give it permissions.  The username will be `services` which is consistent with the provided configuration files.
    - Run `rabbitmqctl add_user services <PASSWORD>` with the password you want to use.  Make sure to remember this password as it will be used when updating configuration files.
    - Run `rabbitmqctl set_permissions -p / services ".*" ".*" ".*"`

**Example Rabbit Cluster Connection Logs**
```
 =INFO REPORT==== 21-Apr-2017::21:05:02 ===
 node 'rabbit@rabbit-server-1.rabbit-service.default.svc.cluster.local' up
 =INFO REPORT==== 21-Apr-2017::21:05:12 ===
 rabbit on node 'rabbit@rabbit-server-1.rabbit-service.default.svc.cluster.local' up
```

### Create ProgMan Configuration
This section assumes there is an existing Progman configuration setup for Student and Proctor.
1. Go to an existing configuration.
1. Set the `Property Editor` to “Property File Entry”
1. Copy the values in the text area.
1. Create a new profile named `tds, tdsuat`
1. Once again set the `Property Editor` to “Property File Entry”
1. Paste the values you copied on step 3 into the textarea and save the configuration.
1. Go back and edit the newly created configuration.  Use the “Property File Entry” mode.
1. Add the following properties at the end of the text area.  
```
tds.exam.remote.enabled=true
tds.exam.remote.url=http://tds-exam-service/exam
tds.exam.legacy.enabled=false
tds.assessment.remote.url=http://tds-assessment-service/
tds.session.remote.url=http://tds-session-service/sessions
tds.session.remote.enabled=true
tds.session.legacy.enabled=false
```

### Update Configuration YML Files 
These files are located in the **tds_deployment_support_files.zip** and should reside in your private Gitlab repository if you followed the previous steps.  After making the edits make sure that you push the changes to your Gitlab private repository on the master branch so the Spring Cloud Config can leverage the values.

Type of information being modified

- DB information
- Redis
- Rabbit
- S3 information in tds-exam-service.yml

There are comments within the configuration files explaining what certain properties represent.  Any property that has `<SOMETHING>` style means that you will need to edit it following the steps we present below.  
Any property that starts with `{cipher}` means that we recommend you secure this value using the Spring CLI tool mentioned above.  The `{cipher}` keyword should remain and the value after to be replaced.  For example, `{cipher}<RABBIT PASSWORD>`, you’d replace the `<RABBIT PASSWORD>` part with the value created via the Sprint CLI.

**Important:** Make sure you remember the encrypt key you use for encryption and make sure to use the same one throughout.  You will also be instructed to add it to a deployment file in a later step.

Example Spring CLI Encrypt command usage

1. Run `spring encrypt my_password --key my_encrypt_key`
1. Resulting in output `604a89112855fc5fd80132c3a6f2a331639a0ffe0c280d65848833276062cebb`
1. Take that value and replace the `{cipher}` property with this value.  So `{cipher}<RABBIT PASSWORD>` would be `{cipher}604a89112855fc5fd80132c3a6f2a331639a0ffe0c280d65848833276062cebb`

### S3 Configuration
In the tds-exam-service.yml file you will see a section like this:
```
    s3:
      bucketName: tds-resources
      itemPrefix: uat/
      accessKey: '{cipher}<REDACTED>'
      secretKey: '{cipher}<REDACTED>'
```
The `accessKey` and `secretKey` should be the AWS’s read only user’s `accessKey` and `secretKey`.  The `bucketName` and `itemPrefix` are as you configured it when you loaded the items to s3.  The values above are the ones used in the section that covered items to S3.

### Update Deployment Configuration Files

This section refers to the files in the deployment directory contained in the `<ZIPFILE>` file.  The information in all files needs to be updated with your local deployment environment.  For example, there are references to Progman (PM) and Single Sign On (SSO) which you should have already configured if you’ve followed previous steps in the deployment checklist.

**Important:** Since these files should never be publicly available nor hosted the passwords should be saved as plain text.

**NOTE:** Any file that has a header `This file requires per-deployment/environment configuration changes` will have to be updated with information concerning your deployment environment.  Otherwise the file does not need to be modified.

#### Proctor Deployment Configuration
The following settings need to be configured in the `tds-proctor-war.yml` file.  These configuration settings are stored in the `env` section of the `yml` file.  Each configuration setting is listed as a name/value pair.  When making changes to the `tds-proctor-war.yml` file, only the value needs to be updated.  In some cases, the value does not need to be edited.

* `GET_HOSTS_FROM`: [*Tell Kubernetes where/how to look up the host names of pods within the Kubernetes environment*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.
* `OAUTH_ACCESS_URL`: [*the url to OpenAM for authentication*{: style="color: #f00;"}]
* `PM_OAUTH_CLIENT_ID`: [*the client ID used to authenticate with OpenAM. Typically this value is “pm”*{: style="color: #f00;"}]
* `PM_OAUTH_CLIENT_SECRET`: [*the “password” for whatever account you have set in the PM_OAUTH_CLIENT_ID*{: style="color: #f00;"}]
* `PM_OAUTH_BATCH_ACCOUNT`: [*In our environments, we have this set to prime.user@example.com*{: style="color: #f00;"}]
* `PM_OAUTH_BATCH_PASSWORD`: [*the password for whatever account is set in the PM_OAUTH_BATCH_ACCOUNT*{: style="color: #f00;"}]
* `PROGMAN_BASEURI`: [*The URL to the ProgMan REST component, e.g. http://progman-example.com/rest/*{: style="color: #f00;"}]
* `PROGMAN_LOCATOR`: [*The ProgMan profile containing the settings for the Proctor application, e.g. **tds,development***{: style="color: #f00;"}]
* `SPRING_PROFILES_ACTIVE`: [*The Spring profiles in use by the Proctor application when it starts up.*{: style="color: #f00;"}]
* `CATALINA_OPTS`: [*The options passed into the Proctor application when the tomcat server starts up*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.  However, they may need to be modified based on the traffic/workload that Proctor is expected to handle.
* `PROCTOR_SECURITY_SAML_ALIAS`: [*this value should be registered in OpenAM*{: style="color: #f00;"}]
* `PROCTOR_SECURITY_SAML_KEYSTORE_CERT`: [*this is the “alias” for the private key entry in the samlKeystore.jks*{: style="color: #f00;"}]
* `PROCTOR_SECURITY_SAML_KEYSTORE_PASS`: [*this is the password to open the samlKeystore.jks*{: style="color: #f00;"}]
* `PROCTOR_WEBAPP_SAML_METADATA_FILENAME`: [*this is the name of the SAML metadata file that should also be in*{: style="color: #f00;"}]
* `RABBITMQ_ADDRESSES`: [*The list of all the RabbitMQ services that are running within the Kubernetes cluster*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.
* `RABBITMQ_USERNAME`:  [*The RabbitMQ user account created for access to the RabbitMQ instance hosted witin the Kubernetes cluster*{: style="color: #f00;"}]
* `RABBITMQ_PASSWORD`:  [*The password for the RabbitMQ user account*{: style="color: #f00;"}]
* `CONFIG_SERVICE_URL`: [*The URL to the Spring Cloud Configuration service within the Kubernetes environment*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.
* `LOGSTASH_DESTINATION`: [*The url of the Logstash server that stores logging information*{: style="color: #f00;"}]

Shown below is an example of a configured `env` section of a `tds-proctor-war.yml` file:

<div class="highlighter-rouge">
  <pre class="highlight">
    <code>
    env:
    - name: GET_HOSTS_FROM
      value: dns
    - name: OAUTH_ACCESS_URL
      value: "<span class="placeholder-example">https://sso-example.org/auth/oauth2/access_token?realm=/sbac</span>"
    - name: PM_OAUTH_CLIENT_ID
      value: "<span class="placeholder-example">pm</span>"
    - name: PM_OAUTH_CLIENT_SECRET
      value: "<span class="placeholder-example">[redacted]</span>"
    - name: PM_OAUTH_BATCH_ACCOUNT
      value: "<span class="placeholder-example">prime.user@example.com</span>"
    - name: PM_OAUTH_BATCH_PASSWORD
      value: "<span class="placeholder-example">[redacted]</span>"
    - name: PROGMAN_BASEURI
      value: "<span class="placeholder-example">http://progman-example.org:8080/rest/</span>"
    - name: PROGMAN_LOCATOR
      value: "<span class="placeholder-example">tds,example</span>"
    - name: SPRING_PROFILES_ACTIVE
      value: "mna.client.null,progman.client.impl.integration,server.singleinstance"
    - name: CATALINA_OPTS
      value: "-XX:+UseConcMarkSweepGC -Xms512m -Xmx1512m -XX:PermSize=256m -XX:MaxPermSize=512m"
    # Note that the values below are used by SAML for SSO.  They should match the values in the associated
    # security files ./security/proctor/*
    - name: PROCTOR_SECURITY_SAML_ALIAS
      value: <span class="placeholder-example">proctorexample</span>
    - name: PROCTOR_SECURITY_SAML_KEYSTORE_CERT
      value: proctor-deploy-sp
    - name: PROCTOR_SECURITY_SAML_KEYSTORE_PASS
      value: <span class="placeholder-example">[redacted]</span>
    - name: PROCTOR_WEBAPP_SAML_METADATA_FILENAME
      value: proctor_sp_deployment.xml
    - name: RABBITMQ_ADDRESSES
      value: "rabbit-server-0.rabbit-service:5672,rabbit-server-1.rabbit-service:5672,rabbit-server-2.rabbit-service:5672"
    - name: RABBITMQ_USERNAME
      value: "<span class="placeholder-example">services</span>"
    - name: RABBITMQ_PASSWORD
      value: "<span class="placeholder-example">[redacted]</span>"
    - name: CONFIG_SERVICE_URL
      value: "http://configuration-service"
    - name: LOGSTASH_DESTINATION
      value: "logstash.sbtds.org:4560"
    </code>
  </pre>
</div>

#### Student Deployment Configuration
The following settings need to be configured in the `tds-proctor-war.yml` file.  These configuration settings are stored in the `env` section of the `yml` file.  Each configuration setting is listed as a name/value pair.  When making changes to the `tds-proctor-war.yml` file, only the value needs to be updated.  In some cases, the value does not need to be edited.

* `GET_HOSTS_FROM`: [*Tell Kubernetes where/how to look up the host names of pods within the Kubernetes environment*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.
* `OAUTH_ACCESS_URL`: [*the url to OpenAM for authentication*{: style="color: #f00;"}]
* `PM_OAUTH_CLIENT_ID`: [*the client ID used to authenticate with OpenAM. Typically this value is “pm”*{: style="color: #f00;"}]
* `PM_OAUTH_CLIENT_SECRET`: [*the “password” for whatever account you have set in the PM_OAUTH_CLIENT_ID*{: style="color: #f00;"}]
* `PM_OAUTH_BATCH_ACCOUNT`: [*In our environments, we have this set to prime.user@example.com*{: style="color: #f00;"}]
* `PM_OAUTH_BATCH_PASSWORD`: [*the password for whatever account is set in the PM_OAUTH_BATCH_ACCOUNT*{: style="color: #f00;"}]
* `PROGMAN_BASEURI`: [*The URL to the ProgMan REST component, e.g. http://progman-example.com/rest/*{: style="color: #f00;"}]
* `PROGMAN_LOCATOR`: [*The ProgMan profile containing the settings for the Student application, e.g. **tds,development***{: style="color: #f00;"}]
* `SPRING_PROFILES_ACTIVE`: [*The Spring profiles in use by the STudent application when it starts up.*{: style="color: #f00;"}]
  * **NOTE:** to enable caching with Redis, add the `redisCaching` provile to the comma-separated list above
* `CATALINA_OPTS`: [*The options passed into the Student application when the tomcat server starts up*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.  However, they may need to be modified based on the traffic/workload that Student is expected to handle.
* `RABBITMQ_ADDRESSES`: [*The list of all the RabbitMQ services that are running within the Kubernetes cluster*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.
* `RABBITMQ_USERNAME`:  [*The RabbitMQ user account created for access to the RabbitMQ instance hosted witin the Kubernetes cluster*{: style="color: #f00;"}]
* `RABBITMQ_PASSWORD`:  [*The password for the RabbitMQ user account*{: style="color: #f00;"}]
* `CONFIG_SERVICE_URL`: [*The URL to the Spring Cloud Configuration service within the Kubernetes environment*{: style="color: #f00;"}]
  * **NOTE:** Typically this will not have to be modified.
* `LOGSTASH_DESTINATION`: [*The url of the Logstash server that stores logging information*{: style="color: #f00;"}]
* `PERFORMANCE_REQUEST_LIMIT`: [*The number of concurrent requests that IRiS is allowed to handle*{: style="color: #f00;"}]

Shown below is an example of a configured `env` section of a `tds-proctor-war.yml` file:

<div class="highlighter-rouge">
  <pre class="highlight">
    <code>
    env:
    - name: GET_HOSTS_FROM
      value: dns
    - name: OAUTH_ACCESS_URL
      value: "<span class="placeholder-example">https://sso-example.org/auth/oauth2/access_token?realm=/sbac</span>"
    - name: PM_OAUTH_CLIENT_ID
      value: "<span class="placeholder-example">pm</span>"
    - name: PM_OAUTH_CLIENT_SECRET
      value: "<span class="placeholder-example">[redacted]</span>"
    - name: PM_OAUTH_BATCH_ACCOUNT
      value: "<span class="placeholder-example">prime.user@example.com</span>"
    - name: PM_OAUTH_BATCH_PASSWORD
      value: "<span class="placeholder-example">[redacted]</span>"
    - name: PROGMAN_BASEURI
      value: "<span class="placeholder-example">http://progman-example.org:8080/rest/</span>"
    - name: PROGMAN_LOCATOR
      value: "<span class="placeholder-example">tds,example</span>"
    - name: SPRING_PROFILES_ACTIVE
      value: "mna.client.null,progman.client.impl.integration,server.singleinstance"
    - name: CATALINA_OPTS
      value: "-XX:+UseConcMarkSweepGC -Xms512m -Xmx1512m -XX:PermSize=256m -XX:MaxPermSize=512m"
    - name: RABBITMQ_ADDRESSES
      value: "rabbit-server-0.rabbit-service:5672,rabbit-server-1.rabbit-service:5672,rabbit-server-2.rabbit-service:5672"
    - name: RABBITMQ_USERNAME
      value: "<span class="placeholder-example">services</span>"
    - name: RABBITMQ_PASSWORD
      value: "<span class="placeholder-example">[redacted]</span>"
    - name: CONFIG_SERVICE_URL
      value: "http://configuration-service"
    - name: LOGSTASH_DESTINATION
      value: "<span class="placeholder-example">logstash.sbtds.org:4560</span>"
    - name: PERFORMANCE_REQUEST_LIMIT
      value: "<span class="placeholder-example">50</span>"
    </code>
  </pre>
</div>

### Cipher Decryption Configuration
You’ll need to use the value you used for `my_encrypt_key` password encryption for the following steps.

1. Navigate to `deployment/configuration-service.yml` which should be wherever you extracted the accompanying zip file.
2. Update the `ENCRYPT_KEY` property value to be your `my_encrypt_key` value
  - This will allow the applications to decrypt the passwords

### HTTPS Communication
The default deployment files assume communication is done with HTTPS. However, if your environment does not support HTTPs you will need to make a couple edits to `tds-ingress.yml` which resides in the `deployment` directory.
Remove both `ingress.kubernetes.io/force-ssl-redirect: "true"` annotations.

### Create TDS Environment with Kubernetes
The following steps will walk you through standing up the TDS environment using Kubernetes.  All the steps below should be run within the `deployment` directory containing the deployment configuration yml files in the previous step.

1. Create the Spring Cloud Configuration Service
  - Run `kubectl create -f configuration-service.yml`
  - Wait for the spring cloud configuration service to initialize.  You can see this by running `kubectl get po -w`.  Once the service is RUNNING at a 1/1 state it means it is initialized.
1. Create the remaining TDS Kubernetes resources
  - Run `cd ..` to change to the parent directory 
  - Run `kubectl create -f deployment` to create the remain kubernetes resources in the `deployment` directory.
  - **Note** You will see “Already Created” errors for kubernetes resources you have created in previous steps.  For example, rabbit, ingress, and configuration service.  
  - Run `kubectl get po -w` and wait until all services are “Running” with a 1/1.  If there are errors please refer to the “Troubleshooting” section at the end of this document.

### Provide External Access
The following steps walk you through setting up the system to be available for public access.

1. Find and keep the auto-generated load balancer name for the nginx ingress service.  
  - Run `kubectl -n kube-system describe svc nginx-ingress-service | grep 'LoadBalancer Ingress'` 
  - Copy and store the id associated with the Ingress ELB (Elastic Load Balancer)
1. Register a CNAME for your host (referenced in your Ingress routes) pointing to the Ingress ELB
  - We use AWS Route53 to manage this information
1. Record the ports that the ELB is forwarding to because they are auto-generated
1. If you’re doing SSL:
  - Register an SSL certificate with the ELB to provide SSL termination at the ELB
  - Change the ELB from TCP/SSL protocols to HTTP/HTTPS forwarding to the same HTTP port that was recorded in step 3.  This ensures the ELB adds the necessary X-Forwarded headers to requests.
  - If you forgot to write those down you can still get the information
	  - Run `kubectl -n kube-system describe svc nginx-ingress-service`
	  - The ports you want are the nodeport for HTTP and HTTPS
	  - The ELB should be set up to do HTTP and HTTPS but by default may be TCP.
1. If you’re **not** doing SSL:
  - Change the ELB from TCP/SSL protocols to HTTP/HTTPS forwarding to the separate ports recorded in step 3.
1. Verify everything is running by navigating a browser to `<YOUR HOST>/student` and `<YOUR HOST>/proctor`

### Configuring Proctor
Once creating a new proctor deployment you will need to do a couple extra steps to configure it to work with OpenAM and SAML needs.  The steps below assume you are using SAML and OpenAM for security.

1. Go to `<Your Host>/proctor/saml/metadata`
	* If things are working correctly this will give you a `spring_saml_metadata.xml` file
2. Log into OpenAM.
	1. Under the `Common Tasks` tab and `Create SAMLv2 Providers` click the `Register Remote Service Provider`
	2. Configure the Provider
		1. Pick the realm you want to use.  Should be single one if you followed the provided deployment steps.
		2. Paste the URL that you typed in on step 1 into `URL where metadata is located` textbox
		3. You should not have to change the Circle of trust as it should be whatever you set up in the OpenAM configuration.
		4. Click `Configure` at the top right.
3. Verify it is configured in OpenAM
	1. 	click the `Federation` tab
	2. You should see a `entity id` + `saml2`
	3. You should have the same `entity id` in the `Entity Providers` section
4. Go to Proctor to see it is configured properly.

{% include checklist/redploy_kubernetes_pod.md %}

### Troubleshooting
The cause of most failures during deployment will be due to configuration issues either in the configuration or deployment yml files.

#### Resolving CIDR Range Conflicts
It is possible that `kops` will report a conflict/overlap between CIDR blocks during setup.  The error will appear similar to this:

```
InvalidSubnet.Conflict: The CIDR 'XXX' conflicts with another subnet
```

**NOTE:** 'XXX' will be the CIDR range that is causing the conflict

To see the existing subnets of your VPC, use the following command:

`aws ec2 describe-subnets --filters Name=vpc-id,Values=`[*the id of the VPC created earlier*{: style="color: #f00;"}]

Example: `aws ec2 describe-subnets --filters Name=vpc-id,Values=vpc-87f620fe`

the output will appear similar to what's shown below:

```
{
    "Subnets": [
        {
            "VpcId": "vpc-87f620fe",
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "pri-sectest.sbtds.org"
                }
            ],
            "CidrBlock": "170.20.1.0/24",
            "AssignIpv6AddressOnCreation": false,
            "MapPublicIpOnLaunch": false,
            "SubnetId": "subnet-e0362186",
            "State": "available",
            "AvailableIpAddressCount": 251,
            "AvailabilityZone": "us-west-2a",
            "Ipv6CidrBlockAssociationSet": [],
            "DefaultForAz": false
        },
        {
            "VpcId": "vpc-87f620fe",
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "pub-sectest.sbtds.org"
                }
            ],
            "CidrBlock": "170.20.0.0/24",
            "AssignIpv6AddressOnCreation": false,
            "MapPublicIpOnLaunch": false,
            "SubnetId": "subnet-c03522a6",
            "State": "available",
            "AvailableIpAddressCount": 249,
            "AvailabilityZone": "us-west-2a",
            "Ipv6CidrBlockAssociationSet": [],
            "DefaultForAz": false
        }
    ]
}
```

Edit the cluster configuration to eliminate the conflicts.  A CIDR range calculator (e.g. [this one](http://www.subnet-calculator.com/cidr.php) or [this one](https://www.ipaddressguide.com/cidr)) may be useful in verifying that the chosen IP address ranges do not overlap

An example of a cluster configuration edited to avoid CIDR conflicts is shown below:

`kops edit cluster sectest.sbtds.org`

<div class="highlighter-rouge">
  <pre class="highlight">
    <code>
      apiVersion: kops/v1alpha2
      kind: Cluster
      metadata:
        creationTimestamp: 2018-01-12T02:04:16Z
        name: sectest.sbtds.org
      spec:
        api:
          loadBalancer:
            type: Public
        authorization:
          alwaysAllow: {}
        channel: stable
        cloudProvider: aws
        configBase: s3://kops-sbtds-org-state-store/sectest.sbtds.org
        dnsZone: sbtds.org
        etcdClusters:
        - etcdMembers:
          - instanceGroup: master-us-east-1a
            name: a
          name: main
        - etcdMembers:
          - instanceGroup: master-us-east-1a
            name: a
          name: events
        iam:
          allowContainerRegistry: true
          legacy: false
        kubernetesApiAccess:
        - 0.0.0.0/0
        kubernetesVersion: 1.8.4
        masterInternalName: api.internal.sectest.sbtds.org
        masterPublicName: api.sectest.sbtds.org
        networkCIDR: 170.20.0.0/16
        networkID: vpc-2348765b
        networking:
          weave:
            mtu: 8912
        nonMasqueradeCIDR: 100.64.0.0/10
        sshAccess:
        - 0.0.0.0/0
        subnets:
        - cidr: 170.20.32.0/19
          name: us-east-1a
          type: Private
          zone: us-east-1a
        - cidr: <span class="placeholder-example">170.20.10.0/22  # <-- this is the change to avoid the conflict; the CIDR range reported in the error was 170.20.0.0/22</span>
          name: utility-us-east-1a
          type: Utility
          zone: us-east-1a
        topology:
          bastion:
            bastionPublicName: bastion.sectest.sbtds.org
          dns:
            type: Public
          masters: private
          nodes: private
    </code>
  </pre>
</div>

Run `kops update cluster sectest.sbtds.org --state s3://kops-sbtds-org-state-store --yes` to update the cluster

#### Handy kubectl commands

- `kubectl get po -w` ←- use this command to follow the creation of the pods
- `kubectl proxy` ← to start the UI; navigate to [http://localhost:8001](http://localhost:8001)
- `kubectl get node` ← to show the nodes that support the 
- `kubectl -n kube-system get po` ← to see all the “system namespace” pods
- `kubectl logs -f [pod name]` ← to stream logs for a particular pod
- `kubectl logs --previous [pod name]`  ← retrieves previous logs for any pod that has been shut down, restarted, or failed. 
- `kubectl exec -it [pod name] /bin/bash` ← to open a bash interactive terminal on the pod
- `kubectl exec -it [pod name] /bin/sh` ← to open a shell interactive terminal on the pod.  Used when the pod doesn’t have bash installed

#### Restarting a Kubernetes Pod
This is useful when pushing a configuration yml update.

- `kubectl get po` ← list all of the pods
- `kubectl delete po [space-delimited list of pod names]` ← deleting the pods. Kubernetes will stand up new pods to replace the deleted ones using the updating configuration information.
- `kubectl get po -w` ← watch the deployment of the new pods. Once everything is running at 1/1 state it means it is complete.

[back to Deployment Checklists](index.html)