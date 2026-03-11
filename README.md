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
- **namespace**: A way to divide cluster resources between multiple users or teams. Namespaces provide a scope for names and can be used to organize and manage resources in a Kubernetes cluster.

### Persistent Volumes and Persistent Volume Claims
<!-- continua... -->

> To learn more about Kubernetes components, you can refer to the official Kubernetes documentation: https://kubernetes.io/docs/concepts/overview/components/
>
>  About Ingress, you can refer to the official Kubernetes documentation: https://kubernetes.io/docs/concepts/services-networking/ingress/

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
- `kubectl api-resources`: Lists all the available API resources in the Kubernetes cluster, which can help you understand the different types of resources you can manage and interact with in your cluster.
- `kubectl api-resource --namespaced=<true|false>`: Lists the API resources that are either namespaced or cluster-scoped, allowing you to filter the resources based on their scope.

> to get namespaced info, for exaple, from a pod, you can use `kubectl get pod <pod-name> -n <namespace-name> -o yaml` to get the detailed information about the pod in a specific namespace.

#### useful sh commands for managing resources

- `kubectl apply -f <file.yaml>`: Applies the configuration defined in a YAML file to create or update Kubernetes resources.
- `kubectl delete -f <file.yaml>`: Deletes the Kubernetes resources defined in a YAML file.
- `kubectl edit deployment <deployment-name>`: Opens the Deployment configuration in a text editor, allowing you to make changes to the Deployment's configuration and save it to apply the changes.

#### minikube specific commands

- `minikube start`: Starts a local Kubernetes cluster using Minikube.
- `minikube stop`: Stops the Minikube cluster.
- `minikube delete`: Deletes the Minikube cluster and all its resources.
- `minikube dashboard`: Opens the Kubernetes Dashboard in a web browser, providing a graphical interface to manage and monitor the Kubernetes cluster.
- `minikube ip`: Displays the IP address of the Minikube cluster, which can be used to access services running in the cluster from your local machine.
- `minikube service <service-name>`: Opens the specified service in a web browser, allowing you to access the application running in the service from your local machine.
- `minikube addons enable ingress`: Enables the Ingress addon in Minikube, allowing you to use Ingress resources to manage external access to services in the cluster.

## More additional knowledge

- **Kubernetes Dashboard**: A web-based user interface for managing and monitoring Kubernetes clusters. It provides a visual representation of the cluster's resources and allows users to perform various operations such as deploying applications, managing resources, and viewing logs and events.
- **Kubernetes API**: The Kubernetes API is the primary interface for interacting with the Kubernetes cluster. It provides a RESTful API that allows users and applications to create, read, update, and delete Kubernetes resources. The API is used by tools like `kubectl` and the Kubernetes Dashboard to manage the cluster and its resources.
- **Kubernetes Operators**: A Kubernetes Operator is a method of packaging, deploying, and managing a Kubernetes application. Operators extend the Kubernetes API to create, configure, and manage instances of complex applications on behalf of a Kubernetes user. They are used to automate the management of applications and services in Kubernetes, making it easier to deploy and maintain complex applications in a Kubernetes cluster.

### Helm

> Official Helm documentation: https://helm.sh/docs/

- **Helm**: Helm is a package manager for Kubernetes that allows you to define, install, and manage Kubernetes applications using Helm charts. A Helm chart is a collection of files that describe a related set of Kubernetes resources. Helm simplifies the deployment and management of applications in Kubernetes by providing a templating mechanism and a way to manage application dependencies. With Helm, you can easily deploy complex applications with a single command and manage their lifecycle using Helm's built-in features for upgrading, rolling back, and uninstalling applications.

#### Helm configuration files

- `Chart.yaml`: This file contains metadata about the Helm chart, such as its name, version, and description. It is used to define the chart and its dependencies.
- `values.yaml`: This file contains the default configuration values for the Helm chart. It allows users to customize the deployment of the application by providing their own values for the configuration parameters defined in the chart.
- `templates/`: This directory contains the Kubernetes resource templates that define the resources to be created when the Helm chart is deployed. These templates can use the values defined in `values.yaml` to customize the resources based on user input.
- `charts/`: This directory can contain other Helm charts that are dependencies of the main chart. It allows you to manage and deploy multiple related applications together as a single unit.
- `LICENSE`: This file contains the license information for the Helm chart, specifying the terms under which the chart can be used and distributed.
- `README.md`: This file provides documentation for the Helm chart, including instructions on how to install and use the chart, as well as any additional information about the application being deployed. It is important to include a README file to help users understand the purpose of the chart and how to use it effectively.

#### Helm commands

- `helm install <release-name> <chart-path>`: Installs a Helm chart with the specified release name and chart path.
- `helm upgrade <release-name> <chart-path>`: Upgrades an existing Helm release with a new chart version or configuration.
- `helm uninstall <release-name>`: Uninstalls a Helm release, removing all associated resources from the Kubernetes cluster.
- `helm list`: Lists all the Helm releases currently deployed in the Kubernetes cluster.
- `helm status <release-name>`: Displays the status of a specific Helm release, including information about the deployed resources and their current state.
- `helm repo add <repo-name> <repo-url>`: Adds a Helm chart repository to the local Helm configuration, allowing you to access and install charts from that repository.
- `helm repo update`: Updates the local Helm chart repository cache, ensuring that you have the latest information about available charts and their versions.
- `helm search repo <chart-name>`: Searches for a specific Helm chart in the configured Helm repositories, allowing you to find and install charts that match your requirements.
- `helm template <chart-path>`: Generates the Kubernetes resource manifests from a Helm chart without actually installing it, allowing you to review the generated resources before deployment.

#### Helm best practices

- Use meaningful release names: Choose descriptive and meaningful release names for your Helm deployments to make it easier to identify and manage them in the Kubernetes cluster.
- Keep values.yaml organized: Organize the configuration values in `values.yaml` in a clear and structured manner, grouping related values together and providing comments to explain their purpose and usage.
- Use version control: Store your Helm charts in a version control system (e.g., Git) to track changes, collaborate with others, and maintain a history of your chart development.
- Test your charts: Use tools like `helm lint` and `helm test` to validate your Helm charts and ensure that they are correctly defined and function as expected before deploying them to a production environment.
- Document your charts: Provide clear and comprehensive documentation in the `README.md` file of your Helm chart, including installation instructions, configuration options, and any additional information that users may need to effectively use the chart. This will help users understand how to deploy and manage the application using your Helm chart.

#### Helm versions

- Helm 2: The original version of Helm, which uses Tiller as the server-side component to manage releases and resources in the Kubernetes cluster. Helm 2 is no longer actively maintained and has been deprecated in favor of Helm 3.
- Helm 3: The current version of Helm, which removes the need for Tiller and operates entirely client-side. Helm 3 provides improved security, better support for Kubernetes features, and a more streamlined user experience compared to Helm 2. It is the recommended version for new Helm deployments and is actively maintained with regular updates and improvements.

