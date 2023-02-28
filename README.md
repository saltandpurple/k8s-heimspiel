# EKS + k8s-Tooling


## Tools - Install guides

### awscli

> https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### kubectl

> https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

### Lens

> https://docs.k8slens.dev/getting-started/install-lens/

### helm

> https://helm.sh/docs/intro/install/

---

## AWS EKS ###

### Managed control plane

- Can be updated with a 'one-click-solution'. Very easy to use, exceedingly reliable.
- EKS console shows k8s internals as well as related AWS resources.
- Well integrated with the other AWS services. 
  - Compute/EC2
    - SecGroups, ENIs all work as expected
  - Loadbalancers, EBS Volumes will be provisioned dynamically via integrated controllers/drivers
    - Abstract resource (eg PersistentVolumeClaim, LoadBalancer) is created in k8s, the controller does the rest
  - IAM
    - Nodegroup Role + Cluster Role
    - Permissions can be provided to individual workloads via ServiceAccount-Annotations or to all workloads via nodegroup role
  - ECR
      - Repository Permissions integrated with IAM
      - All image repositories can be used, ECR is optional

---

## Kubectl ###

Powerful, versatile, easy-to-use CLI. Provides extensive control over resources running on kubernetes as well as the cluster itself. 

### Getting access to the cluster - setting up your kubeconfig

We will retrieve our kubeconfig directly from AWS EKS - therefore we setup access to AWS first:

Add the following to your ~/.aws/config:

    [profile nzc-k8s-profile]
    region = eu-central-1
    aws_access_key_id = ASK ME FOR THIS
    aws_secret_access_key = ASK ME FOR THIS
    
    [profile k8s-workshop]
    source_profile = nzc-k8s-profile
    role_arn = arn:aws:iam::020746439882:role/eks-admin
    role_session_name = k8s-workshop


Now retrieve your kubeconfig via `awscli`.

    aws eks update-kubeconfig --region eu-central-1 --name k8s-workshop --profile k8s-workshop

That's it.

### Basic commands

### Handling kubeconfig

	kubectl config get-contexts		# Show all available contexts
	kubectl config current-context		# Show currently active context
	kubectl config use-context heimspiel	# Switch context to heimspiel

### Creating objects

	kubectl create namespace nginx					# Create a namespace 
	kubectl apply -f nginx/nginx.yaml -n nginx			# Apply a given manifest in the namespace heimspiel-nginx
	kubectl create deployment nginx2 -n nginx --image=nginx:latest	# Shortcut to create a deployment without a manifest

### Describing objects

	kubectl get pods --all-namespaces		# List pods in all namespaces
	kubectl get pods -n nginx -o wide		# List pods in a certain namespace but with more output
	kubectl describe deployment nginx -n nginx	# Get a detailed description of an object

### Scaling a deployment

	kubectl scale deployment nginx -n nginx --replicas=10	# Adjust number of desired replicas of a deployment (works with statefulset just the same)

### Connecting to a container

	kubectl exec nginx-{HASH} -c nginx --tty -i -n nginx -- bash 	# Connect to a running container - find out pod name with kubectl get pods beforehand

### Creating and using port-forwardings

	kubectl port-forward service/nginx -n nginx 8080:80 	# Create local port-forwarding to a service - Port specification is local:remote
	curl localhost:8080					# If this requires an explanation, you're wrong here.

### Deleting objects

	kubectl delete deployment nginx2 -n nginx	# Delete a resource by specifying it directly
	kubectl delete -f nginx/nginx.yaml		# Delete resources via their manifest

There is much more depth to this, obviously. You can fully handle your cluster and all its resources with kubectl.

---

## Helm

### Print out manifest with variables filled in

    helm template mariadb/ --debug -f helm/values.yaml

### Install/upgrade helm release
Release-Name - Chart - Namespace - Value-Overrides (with files or direct inputs) - Create NS if not exists - Install if not exists

    helm upgrade mariadb mariadb/ -n mariadb -f helm/values.yaml --set secret.rootPassword={ROOT_PW} --create-namespace --install  

---

## Lens

- Lens is basically a visualized version of kubectl
- Kubectl is awesome! However, in reality we use it mostly for smaller debugging, ad-hoc deployments and quick status checks. Everything else is done via Lens.
- Overview of different kubernetes resources - no details though
- Show nginx deployment and later MariaDB statefulset 
- Create port-forwarding
- Show secret
- Rescale nginx
- Difference between statefulsets & deployments
- Delete resources

---

# Hacking

- Please make sure to use your own namespace (specified with the `-n` param ) to avoid conflicts!
