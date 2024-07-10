# GitOps Explained! Episode 1
## Declarative Deployments

Given the challenges with imperative processes, it’s not surprising that Kubernetes was designed with a solution in mind. Kubernetes manifests are the declarative representation of the desired state for resources in a cluster.

### Declarative Example
Looking back at the commands in the imperative example, we can compare them to the YAML manifest, which represents the same state.

Again, you want to:
1. Create a Deployment in your Kubernetes cluster.
1. Expose it using a LoadBalancer Service.
1. Scale the Deployment to 3 replicas.
1. Update the image tag for the Deployment.
1. Rollback the change to the Deployment.

#### Create a Deployment
The original imperative command:
```
kubectl create deployment my-app --image=nginx:latest
```

And the equivalent manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8080
```

While the Kubernetes manifest looks much more complex than the imperative command, the manifest is fully descriptive of the state of the Deployment. Unlike the `kubectl` command which has many default options hidden to the user and options specific to the user's environment (e.g. the namespace of the current context).

You can deploy the manifest by running:
```bash
kubectl apply -f my-app/deployment.yaml 
```

Which should yeld:
```
deployment.apps/my-app configured
```

If you run the command again, you'll notice that the output shows `unchanged`. With a declarative definition of the state of the `Deployment` the result will always be the same (unless someone else makes a change to it after you do).
```
deployment.apps/my-app unchanged
```

#### Expose The Deployment
The original imperative command:
```
kubectl expose deployment my-app \
    --type=LoadBalancer \
    --port=80 \
    --target-port=8080
```

And the equivalent manifest:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

You can deploy the manifest by running:
```bash
kubectl apply -f my-app/service.yaml 
```

Which should yeld:
```
service/my-app configured
```

#### Scale the Deployment
The original imperative command:
```
kubectl scale deployment my-app --replicas=3
```

To scale the deployment, we don't need to find the specific command for scaling deployments, rather we can make the change to the replicas field in the manifest and re-apply the manifest.

To make the equivalent change in the declarative manifest for the `Deployment`, you can update the `replicas` value:
```diff
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
   spec:
--   replicas: 1
++   replicas: 3
```

The full manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3  # Updated from 1 to 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8080
```

After making the change in the manifest, you simply can apply the manifest again by running:
```bash
kubectl apply -f my-app/deployment.yaml 
```

#### Update the Deployment
The original imperative command:
```
kubectl set image deployment/my-app nginx=nginx:2.27.0
```

To make the equivalent change in the declarative manifest for the `Deployment`, you can update the `image` value:
```diff
      spec:
        containers:
        - name: nginx
--        image: nginx:latest
++        image: nginx:2.27.0
          ports:
          - containerPort: 8080
```

The full manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:2.27.0  # Updated the image tag.
        ports:
        - containerPort: 8080
```

Again, after making the change in the manifest, you can apply the manifest again by running:
```bash
kubectl apply -f my-app/deployment.yaml 
```

#### Rollback
The original imperative command:
```
kubectl rollout undo deployment/my-app
```

To achieve the equivalent, you need to undo your change in the manifest:
```diff
      spec:
        containers:
        - name: nginx
--        image: nginx:2.27.0
++        image: nginx:latest
          ports:
          - containerPort: 8080
```

The full manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest  # Rollback the image tag.
        ports:
        - containerPort: 8080
```

### Challenges of Declarative Deployments
Kubernetes manifests are a great solution for creating a declarative state for resources in a cluster. However, Kubernetes does not provide an answer for how to keep these manifests stored externally to the cluster in sync with the live state of the cluster. 

Each time you make a change to manifest, you must re-apply it to the cluster using kubectl apply.

The manifests don’t address all of the challenges of imperative processes. They remove the error-prone process of running several commands in a specific order to arrive at the desired state. They reduce the complexity of the cluster's desired state, improving the coordination of changes. They integrate better with scripting and updates using automation.

Rollbacks are easier if you keep a copy of the old manifests before making changes. When the manifests are reapplied, they can address any drift from the last time they were applied.

The challenges of an audit record, coordination, complexity, merging changes, and automation are still unaddressed. This is where GitOps comes in.