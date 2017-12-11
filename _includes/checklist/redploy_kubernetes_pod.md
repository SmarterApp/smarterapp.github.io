## Redeploying a Kubernetes Pod
There are some cases when a specific pod(s) needs to be updated.  For example, when a new version of the Student application is released a new Docker image will be made available for deployment.  Following the steps below, the environment can be updated without having to re-deploy all pods in the environment.

### Deploying a Specific Version of a Kubernetes Pod
To deploy a specific version of a pod, use the following steps:

1. Identify the version of the pod to deploy (e.g. Student 4.1.0)
2. Refer to the **Version Compatibility** matrix to ensure the desired version is compatible with other software deployed in the environment
3. Find the `deployment` YAML file that describes the pod to deploy
4. Find the `image` element in the `deployment` YAML file identified in Step 3
5. Change the value after the `:` of the image name (referred to as a "tag") to include the specific version of the Docker image to deploy
6. Verify the `imagePullPolicy` is set to `Always`, which will ensure the image is pulled from the Docker repository
  * The default behavior is to only pull the Docker image if it does not already exist ([reference](https://kubernetes.io/docs/concepts/containers/images/)).

An example of choosing a specific version (v 4.0.1) of the `tds-student-war.yml` file is shown below.  Note only the relevant parts of the file are shown; the rest of the file has been omitted for clarity:

<div class="highlighter-rouge">
  <pre class="highlight">
    <code>
        # other parts of the tds-student-war.yml file...

        spec:
          containers:
          - name: tds-student-war
            image: fwsbac/student:<span class="placeholder-example">4.0.1</span>
            imagePullPolicy: Always

        # ... the rest of the tds-student-war.yml file
    </code>
  </pre>
</div>

7. Save the changes made to the `deployment` YAML file
8. After the `deployment` YAML file has been updated, the environment can be updated to use the specified version:
  * The existing pods can be deleted via [`kubectl delete po`](https://kubernetes-v1-4.github.io/docs/user-guide/kubectl/kubectl_delete/), causing Kubernetes to re-deploy the missing pods
  * The existing pods can be updated using [`kubectl replace`](https://kubernetes-v1-4.github.io/docs/user-guide/kubectl/kubectl_replace/)