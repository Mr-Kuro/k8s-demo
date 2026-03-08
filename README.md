# k8s-demo

This is a project to study Kubernetes.

The topics below will describe the architecture decisions of the project. It should be updated whenever a new decision is made or an existing decision is changed.

## Context

We decided to use Kubernetes for container orchestration because it is a widely adopted solution that provides features such as automatic scaling, self-healing, and easy management of containerized applications.
We want to study Kubernetes and learn how to use it for container orchestration.

K8s-demo is a project to study Kubernetes. It provides a simple application that can be deployed on Kubernetes clusters.

## Some K8s Components

- **Pods**: The smallest deployable units in Kubernetes, which can contain one or more containers. They are used to run applications and services.
- **Services**: An abstraction that defines a logical set of Pods and a policy by which to access them. Services enable communication between different components of an application and provide load balancing.
- **Deployments**: A higher-level abstraction that manages the lifecycle of Pods. Deployments ensure that a specified number of Pods are running at any given time and can perform rolling updates to update the application without downtime.
- **ConfigMaps**: A Kubernetes object that allows you to store configuration data as key-value pairs. ConfigMaps can be used to decouple configuration from application code, making it easier to manage and update configuration without redeploying the application.
- **Secrets**: Similar to ConfigMaps, but designed to store sensitive information such as passwords, API keys, and certificates. Secrets are encoded and can be used to securely manage sensitive data in Kubernetes applications.
- **Ingress**: An API object that manages external access to services in a Kubernetes cluster, typically HTTP. Ingress can provide load balancing, SSL termination, and name-based virtual hosting to route traffic to the appropriate services based on the request's host or path.
- **Persistent Volumes (PVs)**: A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. PVs are used to provide persistent storage for applications running in Kubernetes.
- **Persistent Volume Claims (PVCs)**: A request for storage by a user. PVCs are used to claim storage from PVs and can specify the desired size and access modes for the storage.
- **Namespaces**: A way to divide cluster resources between multiple users or teams. Namespaces provide a scope for names and can be used to organize and manage resources in a Kubernetes cluster.
- **StatefulSets**: A Kubernetes controller that manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods. StatefulSets are used for applications that require stable, unique network identifiers and persistent storage, such as databases.

## Our K8s Components Overview

**Internal Service:**

- Mongo DB: A NoSQL database that provides high performance, high availability, and easy scalability. It is used to store data for our application.

**External Service:**

- Mongo Express: A web-based MongoDB admin interface that allows us to manage our MongoDB database through a user-friendly interface. It provides features such as data visualization, query execution, and database management.

**ConfigMaps:**

- MongoDB URL: A ConfigMap that stores the URL for connecting to the MongoDB database. This allows us to easily update the database connection information without modifying the application code.

**Secrets:**

- MongoDB Credentials: A Secret that stores the username and password for connecting to the MongoDB database. This ensures that sensitive information is securely stored and can be accessed by the application when needed.
  - Username:
  - Password:

**Deployments:**

- MongoDB Deployment: A Deployment that manages the lifecycle of the MongoDB Pods. It ensures that a specified number of MongoDB Pods are running at any given time and can perform rolling updates to update the database without downtime.
- Mongo Express Deployment: A Deployment that manages the lifecycle of the Mongo Express Pods. It ensures that a specified number of Mongo Express Pods are running at any given time and can perform rolling updates to update the application without downtime.

**Graphical Representation:**

```plaintext
+------------------+          +------------------+
|   MongoDB        |          |   Mongo Express  |
|   Deployment     |          |   Deployment     |
|   (Internal)     |          |   (External)     |
+------------------+          +------------------+
|   ConfigMap      |          |   ConfigMap      |
|   (MongoDB URL)  |          |   (MongoDB URL)  |
+------------------+          +------------------+
|   Secret         |          |   Secret         |
|   (MongoDB       |          |   (MongoDB       |
|   Credentials)   |          |   Credentials)   |
+------------------+          +------------------+
```

This architecture allows us to manage our MongoDB database and Mongo Express application efficiently while keeping sensitive information secure. The use of Deployments ensures that our applications are highly available and can be updated without downtime, while ConfigMaps and Secrets provide a flexible way to manage configuration and sensitive data.

## Browser Request Flow

1. A user sends a request to access the Mongo Express web interface through their browser.
2. The request is routed to the Ingress controller, which is responsible for managing external access to the services in the Kubernetes cluster.
3. The Ingress controller routes the request to the Mongo Express Service based on the defined routing rules.
4. The Mongo Express Service forwards the request to one of the Mongo Express Pods managed by the Mongo Express Deployment.
5. The Mongo Express application running in the Pod processes the request and retrieves the necessary data from the MongoDB database using the connection information stored in the ConfigMap and the credentials stored in the Secret.
6. The Mongo Express application generates a response based on the retrieved data and sends it back to the user's browser through the Ingress controller, which then displays the Mongo Express web interface with the requested information.

**Graphical Representation:**

```plaintext

┌──────────────┐
│  web request │
└──────────────┘
        │
        ▼
┌──────────────┐
│   Ingress    │
│  Controller  │
└──────────────┘
        │
        ▼
┌──────────────┐
│ Mongo Express│
│   Service    │
└──────────────┘
        │
        ▼
┌──────────────┐     ┌──────────────┐
│ Mongo Express│────▶│ ConfigMap    │
│   Pod        │ ┐   │ (MongoDB URL)│
└──────────────┘ ▼   └──────────────┘
        │      ┌──────────────────────┐
        │      │        Secret        │
        │      │ (MongoDB Credentials)│s
        |      |  Password: ********* │
        │      │  Username: ********* │
        │      └──────────────────────┘
        ▼
┌──────────────┐
│ MongoDB      │
│ Deployment   │
└──────────────┘
```

## Stack used to study Kubernetes

- **Minikube**: A tool that allows you to run a single-node Kubernetes cluster on your local machine. It is used for development and testing purposes.
- **kubectl**: The command-line tool for interacting with Kubernetes clusters. It is used to deploy applications, manage cluster resources, and view logs and events.
- **Docker**: A platform for developing, shipping, and running applications in containers. It is used to create and manage container images for our application.

### useful sh commands

#### useful sh commands for listing resources

- `kubectl get pods`: Lists all the Pods in the current namespace.
- `kubectl get services`: Lists all the Services in the current namespace.
- `kubectl get deployments`: Lists all the Deployments in the current namespace.
- `kubectl get configmaps`: Lists all the ConfigMaps in the current namespace.
- `kubectl get secrets`: Lists all the Secrets in the current namespace.

##### useful sh commands for debugging and monitoring

- `kubectl describe pod <pod-name>`: Provides detailed information about a specific Pod, including its status, events, and resource usage.
- `kubectl logs <pod-name>`: Retrieves the logs from a specific Pod, which can be useful for debugging and monitoring the application running in the Pod.
- `kubectl exec -it <pod-name> -- /bin/bash`: Opens an interactive terminal session inside a specific Pod, allowing you to run commands and inspect the environment of the container running in the Pod.
- `kubectl top pod <pod-name>`: Displays the resource usage (CPU and memory) of a specific Pod, which can help you monitor the performance of your application and identify potential issues.

#### useful sh commands for managing resources

- `kubectl apply -f <file.yaml>`: Applies the configuration defined in a YAML file to create or update Kubernetes resources.
- `kubectl delete -f <file.yaml>`: Deletes the Kubernetes resources defined in a YAML file.
- `kubectl edit deployment <deployment-name>`: Opens the Deployment configuration in a text editor, allowing you to make changes to the Deployment's configuration and save it to apply the changes.
