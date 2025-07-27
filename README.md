# Kubernetes
Install  minikube on codespace or ec2 machines:
https://github.com/akhileshmishrabiz/Devops-zero-to-hero/blob/main/kubernetes/minikube-setup.md


# Kubernetes Basics Exercises
## Pods, Deployments & Services on Minikube

### Prerequisites
- Minikube installed and running
- kubectl configured to work with minikube
- Basic understanding of YAML syntax

---

## Exercise 1: Working with Pods

### 1.1 Create Your First Pod
**Task**: Create an nginx pod using imperative commands

```bash
# Create a pod
kubectl run nginx-pod --image=nginx --port=80

# Verify the pod is running
kubectl get pods

# Get detailed information about the pod
kubectl describe pod nginx-pod
```

**Expected Output**: Pod should be in "Running" state

### 1.2 Pod Troubleshooting
**Task**: Create a pod with a non-existent image and troubleshoot

```bash
# Create a pod with wrong image
kubectl run broken-pod --image=nginx:non-existent

# Check pod status
kubectl get pods

# Investigate the issue
kubectl describe pod broken-pod
kubectl logs broken-pod
```

**Questions**:
- What is the pod status?
- What error message do you see?
- How would you fix this issue?

### 1.3 Pod YAML Export
**Task**: Export the running nginx-pod to YAML

```bash
# Export pod to YAML
kubectl get pod nginx-pod -o yaml > nginx-pod.yaml

# Clean up the exported YAML (remove status, metadata clutter)
# Edit the file manually or use grep to filter
```

**Challenge**: Create a new pod from the exported YAML with a different name

---

## Exercise 2: Working with Deployments

### 2.1 Create a Deployment
**Task**: Create an nginx deployment with 3 replicas

```bash
# Method 1: Imperative command
kubectl create deployment nginx-deployment --image=nginx --replicas=3

# Verify deployment
kubectl get deployments
kubectl get pods -l app=nginx-deployment
```

### 2.2 Deployment YAML
**Task**: Create the same deployment using YAML

Create a file called `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
# Apply the deployment
kubectl apply -f nginx-deployment.yaml
```

### 2.3 Scaling Operations
**Task**: Practice scaling the deployment

```bash
# Scale up to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5

# Scale down to 2 replicas
kubectl scale deployment nginx-deployment --replicas=2

# Verify scaling
kubectl get pods -l app=nginx
```

### 2.4 Rolling Updates
**Task**: Update the nginx image version

```bash
# Update image version
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Check rollout status
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment
```

### 2.5 Deployment Troubleshooting
**Task**: Create a deployment with issues and troubleshoot

```bash
# Create deployment with wrong image
kubectl create deployment broken-deployment --image=nginx:wrong-tag --replicas=3

# Investigate issues
kubectl get deployments
kubectl get pods
kubectl describe deployment broken-deployment
kubectl describe pod <pod-name>
```

**Troubleshooting Steps**:
1. Check deployment status
2. Examine pod events
3. Check container logs
4. Fix the image tag
5. Verify successful rollout

---

## Exercise 3: Working with Services

### 3.1 ClusterIP Service (Private)
**Task**: Create a private service for nginx deployment

Create `nginx-private-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-private-service
  labels:
    app: nginx
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

```bash
# Apply the service
kubectl apply -f nginx-private-service.yaml

# Verify service
kubectl get services
kubectl describe service nginx-private-service
```

### 3.2 Test Private Service
**Task**: Test internal connectivity

```bash
# Create a test pod to access the service
kubectl run test-pod --image=busybox --rm -it --restart=Never -- /bin/sh

# Inside the test pod, run:
wget -qO- http://nginx-private-service:80
nslookup nginx-private-service
exit
```

### 3.3 NodePort Service (Public)
**Task**: Create a NodePort service for external access

Create `nginx-nodeport-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
```

```bash
# Apply the service
kubectl apply -f nginx-nodeport-service.yaml

# Access the service
minikube service nginx-nodeport-service
# Or get the URL
minikube service nginx-nodeport-service --url
```

### 3.4 LoadBalancer Service (Minikube)
**Task**: Create a LoadBalancer service

```bash
# Create LoadBalancer service
kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80 --name=nginx-loadbalancer

# Enable minikube tunnel (run in separate terminal)
minikube tunnel

# Check external IP
kubectl get services nginx-loadbalancer
```

---

## Exercise 4: Complete Scenarios

### 4.1 Full Stack Deployment
**Task**: Deploy a complete application stack

1. Create a deployment with 3 nginx pods
2. Create both ClusterIP and NodePort services
3. Verify internal and external connectivity
4. Scale the deployment
5. Perform a rolling update

### 4.2 Multi-Service Application
**Task**: Deploy frontend and backend services

```bash
# Deploy nginx as frontend
kubectl create deployment frontend --image=nginx --replicas=2
kubectl expose deployment frontend --port=80 --type=NodePort

# Deploy httpd as backend
kubectl create deployment backend --image=httpd --replicas=3
kubectl expose deployment backend --port=80 --type=ClusterIP

# Test connectivity between services
```

---

## Exercise 5: Troubleshooting Scenarios

### 5.1 Pod Failure Scenario
**Task**: Diagnose and fix a failing pod

```bash
# Create a pod with resource constraints
kubectl run resource-pod --image=nginx --dry-run=client -o yaml > resource-pod.yaml
```

Edit the YAML to add impossible resource requests:

```yaml
resources:
  requests:
    memory: "10Gi"
    cpu: "8"
```

```bash
kubectl apply -f resource-pod.yaml
```

**Troubleshooting Steps**:
1. Check pod status
2. Describe the pod
3. Check events
4. Identify the issue
5. Fix resource requests

### 5.2 Service Connection Issues
**Task**: Fix service connectivity problems

```bash
# Create deployment with wrong labels
kubectl create deployment wrong-labels --image=nginx --replicas=2
kubectl label deployment wrong-labels app=wrong-app

# Create service with mismatched selector
kubectl expose deployment wrong-labels --port=80 --selector=app=nginx
```

**Troubleshooting Steps**:
1. Test service connectivity
2. Check service endpoints
3. Verify label selectors
4. Fix the mismatch

### 5.3 Image Pull Errors
**Task**: Resolve image pull issues

```bash
# Create deployment with private/non-existent image
kubectl create deployment image-issue --image=private-registry/nginx:latest

# Troubleshoot and fix
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

## Exercise 6: Advanced Operations

### 6.1 Export and Backup
**Task**: Export all resources to YAML files

```bash
# Export deployment
kubectl get deployment nginx-deployment -o yaml > backup-deployment.yaml

# Export services
kubectl get service nginx-private-service -o yaml > backup-service.yaml

# Export all resources in namespace
kubectl get all -o yaml > backup-all.yaml
```

### 6.2 Resource Management
**Task**: Monitor and manage resources

```bash
# Check resource usage
kubectl top nodes
kubectl top pods

# Set resource limits
kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","resources":{"limits":{"memory":"128Mi","cpu":"100m"}}}]}}}}'
```

### 6.3 Labels and Selectors
**Task**: Practice with labels and selectors

```bash
# Add custom labels
kubectl label pod nginx-pod environment=production
kubectl label deployment nginx-deployment tier=frontend

# Select resources by labels
kubectl get pods -l environment=production
kubectl get all -l tier=frontend

# Update service selector
kubectl patch service nginx-private-service -p '{"spec":{"selector":{"app":"nginx","tier":"frontend"}}}'
```

---

## Exercise 7: Cleanup and Best Practices

### 7.1 Cleanup Resources
**Task**: Clean up all created resources

```bash
# Delete individual resources
kubectl delete pod nginx-pod
kubectl delete deployment nginx-deployment
kubectl delete service nginx-private-service

# Delete multiple resources
kubectl delete deployment,service -l app=nginx

# Delete all resources in namespace
kubectl delete all --all
```

### 7.2 Best Practices Check
**Task**: Review and implement best practices

**Checklist**:
- [ ] All resources have proper labels
- [ ] Deployments specify resource limits
- [ ] Services use appropriate types
- [ ] Image tags are specific (not latest)
- [ ] Naming conventions are consistent
- [ ] Documentation is updated

---

## Assessment Questions

1. **What's the difference between a Pod and a Deployment?**

2. **When would you use ClusterIP vs NodePort vs LoadBalancer services?**

3. **How do you troubleshoot a pod that's stuck in "Pending" state?**

4. **What happens when you scale a deployment from 3 to 1 replicas?**

5. **How do service selectors match pods to services?**

6. **What's the difference between `kubectl run` and `kubectl create deployment`?**

7. **How would you expose a deployment on a specific port externally in minikube?**

8. **What command would you use to see the rollout history of a deployment?**

---

## Bonus Challenges

### Challenge 1: Multi-Container Pod
Create a pod with nginx and a sidecar container that serves logs.

### Challenge 2: Service Discovery
Create two deployments and demonstrate how they can communicate using service names.

### Challenge 3: Rolling Back
Perform a faulty update and practice rolling back to the previous version.

### Challenge 4: Health Checks
Add readiness and liveness probes to your deployments.

---

## Solution Commands Reference

```bash
# Quick reference for common operations
kubectl get all                          # List all resources
kubectl describe <resource> <name>       # Detailed resource info
kubectl logs <pod-name>                  # Pod logs
kubectl exec -it <pod-name> -- /bin/bash # Access pod shell
kubectl port-forward <pod-name> 8080:80  # Port forwarding
kubectl apply -f <file.yaml>             # Apply YAML file
kubectl delete -f <file.yaml>            # Delete from YAML file
minikube service <service-name>          # Access service in minikube
minikube dashboard                       # Open Kubernetes dashboard
```
