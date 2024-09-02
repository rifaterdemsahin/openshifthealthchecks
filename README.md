# Enabling and Disabling Health Checks in OpenShift

When working with OpenShift, there are times when you need to deploy a module without triggering the usual health check promotion pipeline. This might be necessary for various reasons, such as performing maintenance or updating the application with zero downtime. In this guide, we’ll cover how to temporarily disable the production module health check promotion pipeline to facilitate a deployment with zero replicas.

## Why Disable Health Check Promotion?

By default, OpenShift uses health checks to ensure that each module is running correctly before promoting it to production. However, there are scenarios where you may want to bypass these checks temporarily, such as when:

- Deploying updates or patches that don't require immediate scaling.
- Performing maintenance tasks that necessitate starting with zero replicas.
- Preparing an environment for a controlled deployment where immediate promotion isn't desired.

## Steps to Disable Health Check Promotion

1. **Access Your OpenShift Console**  
   Begin by logging into your OpenShift console. Ensure you have the appropriate permissions to modify deployment configurations.

2. **Navigate to the Deployment Configuration**  
   Find the deployment configuration (DC) of the module you wish to modify. This can typically be found under the "Workloads" section in the OpenShift console.

3. **Edit the Deployment Configuration**  
   Click on the module's deployment configuration to edit it. You'll need to adjust the settings to temporarily disable health checks.

4. **Set the Replicas to Zero**  
   Adjust the replicas field in your deployment configuration to `0`. This ensures that no pods are running during the initial stage of deployment.

5. **Disable Health Check Promotion**  
   Locate the health check settings in the deployment configuration. These are often found under the "Health Checks" tab or section. Temporarily disable the readiness and liveness probes to prevent the deployment pipeline from promoting the module based on health checks.

6. **Apply Changes**  
   Save and apply your changes to the deployment configuration. This will update the configuration and disable the health check promotion for this module.

7. **Deploy Your Module**  
   With health checks disabled and replicas set to zero, you can now deploy your module without triggering the health check promotion pipeline. This setup allows for controlled updates or maintenance without the usual checks and balances.

8. **Re-enable Health Checks After Deployment**  
   Once your deployment or maintenance task is complete, remember to re-enable the health checks and adjust the replicas back to the desired number to resume normal operations.

## Example: Creating a Deployment with Health Checks

To create a Kubernetes Deployment with 3 replicas based on a specific Pod specification, use the following YAML configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: golang-ex-deployment
  namespace: myapp-project
spec:
  replicas: 3
  selector:
    matchLabels:
      app: golang-ex
  template:
    metadata:
      labels:
        app: golang-ex
    spec:
      containers:
        - name: golang-ex
          image: 'image-registry.openshift-image-registry.svc:5000/myapp-project/golang-ex@sha256:5285855b739c8613fa614a9ac991b7e70164bb4db819b0f327a42937a3b9dac2'
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
