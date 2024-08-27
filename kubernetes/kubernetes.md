# Kubernetes: Up and Running by Brendan Burns, Joe Beda, Kelsey Hightower, and Lachlan Evenson 2022

## Creating and Running Containers

The traditional methods of running multiple programs on a single machine
require that all of these programs share the same versions of shared libraries
on the system.
Docker, the default tool most
people use for containers makes it easy to package an executable and push
it to a remote registry where it can later be pulled by others.
Container images bundle a program and its dependencies into a single
artifact under a root filesystem. The most popular container image format is
the Docker image format.

### Container Images

A container image is a binary package that
encapsulates all of the files necessary to run a program inside of an OS
container.

The most popular and widespread container image format is the Docker
image format, which was developed by the Docker open source project for
packaging, distributing, and running containers using the docker
command.

Each layer adds, removes, or modifies files from the preceding layer in the filesystem.

Containers fall into two main categories:

* System containers: seek to mimic virtual machines and often run a full boot
process. They often include a set of system services typically found in a
VM, such as ssh, cron, and syslog

* Application containers: they commonly run a single program

### Building Application Images with Docker

#### Dockerfiles

A Dockerfile can be used to automate the creation of a Docker container image.

The Dockerfile is a recipe for how to build the container image, while .dockerignore defines the set of files that should be ignored when copying files into the image.

```
# Start from a Node.js 16 (LTS) image 
FROM node:16
# Specify the directory inside the image in which all commands will
run 
WORKDIR /usr/src/app
# Copy package files and install dependencies 
COPY package*.json ./
RUN npm install
RUN npm install express
# Copy all of the app files into the image 
COPY . .
# The default command to run when starting the container 
CMD [ "npm", "start" ]
```

Run the following command to create the simple-node Docker image:

```
docker build -t simple-node .
```

To run this image:

```
docker run --rm -p 3000:3000 simple-nod
```

Optimizing Image Sizes:

* files that are removed by subsequent layers in the system are actually still present in the images.
* preceding layers means that they need to be rebuilt, repushed, and repulled to deploy your image to development. In general, you want to order
your layers from least likely to change to most likely to change in order to optimize the image size for pushing and pulling.
* do not do the actual program compilation as part of the construction of the application container image. The
trouble with doing this is that it leaves all of the unnecessary development tools, which are usually quite large

Image Security: don’t build containers with passwords baked in

### Storing Images in a Remote Registry

The standard within the Docker community is to store Docker images in a remote registry.

```
docker login

# tag the 'kuard' image by prepending the target Docker registry. Append another identifier that is usually used for the version of that image, separated by a colon (:)

docker tag kuard gcr.io/kuar-demo/kuard-amd64:blue

# push image to registry

docker push gcr.io/kuar-demo/kuard-amd64:blue

```

### The Container Runtime Interface

Kubernetes provides an API for describing an application deployment, but
relies on a container runtime to set up an application container using the
container-specific APIs native to the target OS.
The interface to this container runtime is defined by the Container Runtime Interface (CRI) standard.
When you install the docker tooling the containerd runtime is also installed and used by the Docker daemon.

Running Containers with Docker:

```
docker run -d --name kuard \
 --publish 8080:8080 \
 gcr.io/kuar-demo/kuard-amd64:blue
 ```

Docker provides the ability to limit the amount of resources used by
applications by exposing the underlying cgroup technology provided by the
Linux kernel. These capabilities are likewise used by Kubernetes to limit
the resources used by each Pod.

Limiting memory resources: use the --memory and --memory-swap flags

```
docker run -d --name kuard \
 --publish 8080:8080 \
 --memory 200m \
 --memory-swap 1G \
 gcr.io/kuar-demo/kuard-amd64:blue
 ```

Limiting CPU resources: Restrict CPU utilization using the --cpu-shares flag

```
 docker run -d --name kuard \
 --publish 8080:8080 \
 --memory 200m \
 --memory-swap 1G \
 --cpu-shares 1024 \
 gcr.io/kuar-demo/kuard-amd64:blue
```

## Deploying a Kubernetes Cluster

### Kubernetes components

* controller-manager: is responsible for running various controllers that regulate behavior in the cluster; for example, ensuring that all of the
replicas of a service are available and healthy.
* scheduler: is responsible for placing different Pods onto different nodes in the cluster.
* etcd server: is the storage for the cluster where all of the API objects are stored
* nodes: nodes are separated into control-plane nodes that contain containers like the API server, scheduler, etc., which manage the cluster,
and worker nodes where your containers will run

### The Kubernetes Client

The official Kubernetes client is kubectl: a command-line tool for interacting with the Kubernetes API

Commands:

```
# check the version of the cluster
kubectl version

# verify that your cluster is generally healthy
kubectl get componentstatuses

# list out all of the nodes in your cluster
kubectl get nodes

# get more information about a specific node
kubectl describe nodes kube1

# label kubernetes objects
kubectl label pods bar color=red

# see logs
kubectl logs <pod-name>

# execute command in running container
kubectl exec -it <pod-name> -- bash

# see kubernetes events
kubectl get events

# top command
kubectl top nodes
kubectl top pods

```

### Cluster Components

* Kubernetes Proxy: responsible for routing network traffic to load-balanced services in the Kubernetes cluster. To do its job, the proxy must be
present on every node in the cluster.

* Kubernetes DNS: provides naming and discovery for the services that are defined in the cluster

* Kubernetes UI: 

### Namespaces

Kubernetes uses namespaces to organize objects in the cluster. You can think of each namespace as a folder that holds a set of objects.

### Creating, Updating, and Destroying Kubernetes Objects

Objects in the Kubernetes API are represented as JSON or YAML files

You can use kubectl to create (or update) this object in Kubernetes by running:

```
kubectl apply -f obj.yaml
```

If you want to see what the apply command will do without actually making the changes, you can use the --dry-run flag to print the objects
to the terminal without actually sending them to the server.

## Pods

A Pod is a collection of application containers and volumes running in the same execution environment. Pods, not containers, are the smallest
deployable artifact in a Kubernetes cluster. This means all of the containers in a Pod always land on the same machine.

Applications running in the same Pod share the same IP address and port
space (network namespace), have the same hostname (UTS namespace),
and can communicate using native interprocess communication channels.

In general, the right question to ask yourself when designing Pods is, “Will
these containers work correctly if they land on different machines?” If the
answer is “no,” a Pod is the correct grouping for the containers.

Create a Pod:

```
kubectl run kuard --generator=run-pod/v1 \
 --image=gcr.io/kuar-demo/kuard-amd64:blue

# check
kubectl get pods
```

### Creating a Pod Manifest:

Pod manifests include a couple of key fields and attributes: 
* metadata section for describing the Pod and its labels,
* spec section for describing volumes and a list of containers that will run in the Pod

```
apiVersion: v1
kind: Pod
metadata:
    name: kuard
spec:
    containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      ports:
      - containerPort: 8080
        name: http
        protocol: TCP
```

### Accessing Your Pod

#### Using Port Forwarding

```
kubectl port-forward kuard 8080:8080
```

A secure tunnel is created from your local machine, through the Kubernetes master, to the instance of the Pod running on one of the worker nodes.

#### Getting More Info with Logs

The kubectl logs command downloads the current logs from the running instance:

```
kubectl logs kuard
```

#### Running Commands in Your Container with exec

 execute commands in the context of the container itself:

 ```
kubectl exec -it kuard ash
 ```

### Health Checks

When you run your application as a container in Kubernetes, it is
automatically kept alive for you using a process health check. This health
check simply ensures that the main process of your application is always
running. If it isn’t, Kubernetes restarts it.

### Liveness Probe

Liveness health checks run application-specific logic, like loading a web page, to verify that the application is not just still running, but is
functioning properly.

Liveness determines if an application is running properly.

Liveness probes are defined per container

```
apiVersion: v1
kind: Pod
metadata:
    name: kuard
spec:
    containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      livenessProbe:
        httpGet:
            path: /healthy
            port: 8080
        initialDelaySeconds: 5      # will not be called until 5 seconds after all the containers in the Pod are created
        timeoutSeconds: 1           # The probe must respond within the 1-second timeout
        periodSeconds: 10           # Kubernetes will call the probe every 10 seconds
        failureThreshold: 3         # If more than 3 consecutive probes fail, the container will fail and restart
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

### Readiness Probe

Readiness describes when a container is ready to serve user requests. Containers that fail readiness checks are removed from service load balancers

### Resource Management

Kubernetes allows users to specify two different resource metrics. Resource
requests specify the minimum amount of a resource required to run the
application. Resource limits specify the maximum amount of a resource that
an application can consume.

#### Minimum Required Resources:

Kubernetes guarantees that these resources are available to the Pod. The
most commonly requested resources are CPU and memory, but Kubernetes
supports other resource types as well, such as GPUs.

```
apiVersion: v1
kind: Pod
metadata:
    name: kuard
spec:
    containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      resources:
        requests:
            cpu: "500m"
            memory: "128Mi"
        limits:
            cpu: "1000m"
            memory: "256Mi"

```

The Kubernetes scheduler will ensure that the sum of all requests of all Pods on a node does
not exceed the capacity of the node

#### Capping Resource Usage with Limits

You can also set a maximum on a it’s resource usage via resource limits.

When you establish limits on a container, the kernel is configured to ensure
that consumption cannot exceed these limits.

### Persisting Data with Volume

Using Volumes with Pods

```
apiVersion: v1
kind: Pod
metadata:
    name: kuard
spec:
    volumes:
    - name: "kuard-data"
      hostPath:
      path: "/var/lib/kuard"
    containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      volumeMounts:
        - mountPath: "/data"
          name: "kuard-data"

```

There are a variety of ways you can use data in your application:

* Communication/synchronization: shared volume between two containers, forming the basis for communication between them
* Cache: An application may use a volume that is valuable for performance
* Persistent data: Sometimes you will use a volume for truly persistent data—data that is
independent of the lifespan of a particular Pod, and should move
between nodes in the cluster if a node fails or a Pod moves to a different
machine for some reason. To achieve this, Kubernetes supports a wide
variety of remote network storage volumes, including widely supported
protocols like NFS and iSCSI as well as cloud provider network storage
like Amazon’s Elastic Block Store
* Mounting the host filesystem

## Labels and Annotations



# Kubernetes Security and Observability by Brendan Creane & Amit Gupta 2021

# Kubernetes Backup & Recovery by Steve Kaelble 2021

