# GitOps Explained! Episode 1
## Imperative Deployments

In an imperative deployment process, you manually execute commands to achieve the desired state of your application or infrastructure. This approach involves direct interaction with the system through command-line interfaces or other tools.

### Imperative Example
You want to:
1. Create a Deployment in your Kubernetes cluster.
1. Expose it using a LoadBalancer Service.
1. Scale the Deployment to 3 replicas.
1. Update the image tag for the Deployment.
1. Rollback the change to the Deployment.

#### Create a Deployment
You begin by creating a deployment using kubectl create.

```
kubectl create deployment my-app --image=nginx:latest
```

You specify the desired number of replicas, the image name and tag. 

#### Expose The Deployment
You need to creating a service to expose the deployment, making it accessible to the outside world.

```
kubectl expose deployment my-app \
    --type=LoadBalancer \
    --port=80 \
    --target-port=8080
```

#### Scale the Deployment
To scale the application, you need to run another command to change the number of replicas.

```
kubectl scale deployment my-app --replicas=3
```

#### Update the Deployment
To update the image tag you manually run a kubectl command.

```
kubectl set image deployment/my-app nginx=nginx:2.27.0
```

#### Rollback
The update introduces an invalid image tag because of a typo! You can roll back to a previous state by executing another kubectl command.

```
kubectl rollout undo deployment/my-app
```

### Challenges of Imperative Deployments
Imperative commands do not provide a way to track changes over time. This lack of version control makes it difficult to:
- **Audit Changes**: There is no built-in history of when a change was made, why it was made, and by whom.
- **Roll Back**: Reverting to a previous configuration (desired state) requires remembering and re-executing previous commands, which can be error-prone or nearly impossible.

Manually executing commands for each change increases operational overhead:
- **Repetition**: Operators must remember and repeat commands accurately, which can lead to potential inconsistencies. You have to remember the precise command to accomplish what you want. Mistyping or copy-and-pasting of commands can lead to serious unintended consequences. Organizations commonly write and maintain scripts to capture and run the commands, but the imperative nature of these scripts makes them difficult to maintain over time.
- **Documentation**: Ensuring all team members know and use the same commands requires constant communication. Each action must be meticulously documented outside the system, often leading to discrepancies between documentation and the actual state. Even worse, documentation requires human interpretation to determine what commands to run, leading to inconsistencies in execution. 

Managing a large number of resources manually becomes impractical as the environment grows:
- **Complexity**: As the number of Kubernetes resources you manage grows, imperative commands become complex and labour-intensive. The commands to target the correct groups of resources to make changes are complicated and difficult to understand. 
- **Coordination**: Coordinating changes to resources across multiple team members and environments is challenging without a unified understanding of the current desired state and the results of the proposed changes.
- **Disaster Recovery**: When the state of the cluster has been established using imperative commands, and the cluster needs to be rebuilt, it’s extremely difficult to return the cluster to the desired state.

Imperative commands can lead to inconsistencies across different environments:
- **Drift**: Environments can drift over time if manual commands are not executed consistently.
- **Environment Differences**: Different team members might use slightly different commands or parameters, leading to subtle differences in configurations.
- **Merging Changes**: Combining changes from multiple sources is more difficult without a central repository to manage conflicts and merges.

The imperative approach does not lend itself well to automation:
- **Integration with CI/CD**: Integrating imperative commands into Continuous Integration/Continuous Deployment (CI/CD) pipelines is more complex and error-prone.