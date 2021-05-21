**Deployment Models**
---------------------

### Bare Metal

*   No restriction on the resources an application could use
    *   Application can hog the entire hardware resource
*   Application running on the same hardware have limited security boundaries

### **Hardware Virtualization**

*   can run multiple VM on the same hardware instance
*   allow flexible scale up or scale down of VMs
*   provided stronger isolation among applications running on the same hardware

### **OS Virtualization**

*   containers are lightweight, easy to create and destroy
*   only one instance of the OS is running
*   each container thinks it have its own OS, but in reality OS is shared between containers

**Isolation and OS Security Features**
--------------------------------------

### **Namespaces**

*   **Linux kernel feature that provides isolation of processes**
*   default namespace: system resources are shared among processes
*   custom namespace: processes in a namespace cannot "see" resources in other namespaces
*   list of namespaces:
    *   PID - Process IDs
    *   IPC - Inter-Process Communication
    *   NET - Network Access
    *   UTS - Hostname and Domain
    *   MNT - File System
    *   USR - Usernames and IDs

### **Control Groups (cgroups)**

*   **Provide resource limits**
*   Setting usage limits on resources (e.g. CPU and memory)
*   Preventing denial-of-service(DOS) attacks

### System Call: Capability Add or Drop

*   **Allow dropping system calls to a predefined list**
*   Principle of least privilege: limiting containers' capabilities
*   Linux capabilities: set of privileges that can be independently enabled or disabled 

### Loadable Security Modules

*   **Provide fine grain access control to resources**
*   Controlling processes' access to system resources
*   Mandatory access control (MAC): SELinux and AppArmor  
    *   not part of kernel code, offered by Loadable Security Modules (LSM) for Linux Kernel

### Seccomp

*   **Limits which system calls a process in user space can execute**
*   Transition a process to a secure state, limiting its capabilities
*   reduces the Linux Kernel attack surface

**Container** **Runtime**
-------------------------

### **High-Level Container Runtime**

*   Responsible for transport, management, packing and unpacking of images

### **Low-Level Container Runtime**

*   Responsible for running the container
*   uses OS features such as namespaces and cgroups to create an isolate containers 

**Kubernetes (K8s)**
--------------------

### Pods

*   All containers in a pod are collocated on a single node
*   Containers in a pod share the same OS context
*   Possible to add isolation to containers within a pod

### Master Node

*   Manages and orchestrates the entire Kubernetes systems

#### etcd

*   distributed key-value-based data store
*   stores the configuration and state of the cluster

#### API Server

*   Accepting and processing commands from clients
*   Components from master and worker node talk to each other via API server

#### Controller Manager

*   Controller: K8s component that watches the state of the cluster and attempts the modify the actual state of the cluster so it matches the desired state
    *   Examples: ReplicaSet, DaemonSet, StatefulSet
*   Watch the API server for state changes

#### Scheduler

*   Uses algorithms to determine which node a pod will be assigned to
*   Posts updates to API server

### Worker Node

*   where containerized applications are run

#### Kubelet

*   Watches for commands posted on API server
*   Runs pods on its node
*   Leverages container runtime
*   Monitors containers running in the pods

#### Kube-Proxy

*   Ensures connectivity to pods
*   Uses OS packet filtering system

#### Container Runtime

*   Kubelet relies on container runtime
*   K8s supports multiple container runtimes

### Kubernetes Objects

*   Abstractions that K8s uses to capture the state of a cluster
*   desired state that the cluster should convert towards
    *   captured in a YAML format

**Five Factors Security Model**
-------------------------------

### Securing Containerized Application Code

*   Finding security vulnerabilities early
*   Embedding security in all stages of SDLC
*   Apply the following design principles:  
    *   Least privilege
    *   Defense in depth
    *   Separation of duties
    *   Minimizing attack surface
*   Static Analysis Security Testing (SAST): Identifying security issues by analyzing source code.
    *   Write secure code in first place
    *   Use scanners' output to locate unsafe code blocks
    *   Complete the feedback loop
*   Software composition analysis: Identifying security issues in open-source and commercial software components
*   Dynamic Application Security Testing (DAST): testing an application at runtime

### Securing Images

*   Protecting images and image registry at every stage
*   Configuration defects in images
*   Ways to minimize attack surface:
    *   Start from a trusted and minimal base image
        *   The **FROM** statement at the top of the Dockerfile
    *   Minimum number of components
        *   Use **ADD** and/or **COPY** to add additional contents to images
    *   Grant Minimum Privileges
        *   No blanket privileges beyond subject's role
        *   Specify user instruction in Dockerfile. Without instruction, containers runs as a root
*   Image Signing
    *   Allows author to add digital signature as its identity 

### Securing Hosts and Container Working Environment

*   Containing running as root **IS NOT EQUAL TO** Privileged container
*   Don't run containers in privileged mode
*   Network policies control what traffic can flow between containers/pods
*   Network drivers: provide communication among containers/pods
    *   bridge - allow containers with the same host to communicate
    *   none - used for closed container, communication will be in loopback mode
    *   host - containers have the same access to the network as the host
    *   underlay - help containers on different hosts communicate using the underlying physical interface
    *   overlay - help containers on different hosts communicate using a virtual network
        *   Virtual network sits above host networks
*   Accept connection on specific interface
*   Minimize Host OS attack surface
    *   Run Minimal OS that has been custom build to run containers
    *   Harden Existing OS but use special harden techniques to reduce attack surface
*   Minimize Direct Access to Host
    *   Let K8s run and manage your containers
    *   treat your hosts as cattle, not pets

### Securing Applications in Kubernetes

#### Securing Applications

*   Discovering and fixing misconfigurations
*   Securing the communications between the K8s core components
*   ensure components are not granted extraordinary privileges

#### Access Management

*   client must prove who it claims to be
*   client must be permitted to perform the action it is request
*   requested operation must be compliant with the security policy
*   API servers uses the following access control plugins to verify the above criteria:
    *   Authentication plugins
    *   Authorization plugins
    *   Admission Control plugins

#### Authenticating Users

*   K8s supports two types of subjects: user/human accounts, and service/machine accounts
*   Support the following authentication methods:
    *   Static Password and Token File
    *   X.509 Client Certificates
    *   OpenID Connect
    *   Service Account Tokens

#### Authenticating Service Accounts

*   Authenticating applications using _ServiceAccount_ resource 
*   Each pod is assigned a _ServiceAccount_ by default
*   _ServiceAccount_ allows controlling which resources a pod can access

#### Authorization

*   Once authenticated, API server validates whether the requested operation is permitted
*   By default, all permission are denied
*   authorization plugin checks for permission only at the verb level
    *   Verbs: create, update. delete and get
    *   More granular validation beyond the verbs is done by the admission controller
*   Role-Based Access Control (RBAC)
    *   Grant or deny access to operations based on subject's role

#### Admission Control

*   Intercepts request and validates it against policies
*   Plugin examples:
    *   AlwaysAdmit
        *   setting all requests to be allowed / disabling admission control
    *   AlwaysDeny
        *   setting all requests to be denied
    *   NamespaceLifecycle
        *   stop pod creation in the namespace if the namespace is in the process of being terminated
    *   ResourceQuota
        *   stop request that violates the resource contraints specified in the ResourceQuota object 

#### Security Context

*   Mechanism provided to developers
*   Can attach the security context for a pod itself when writing pod sepcifications

#### Security Policy

*   Cluster level mechanism for admin to ensure any non-security compliant requests are denied
*   Implemented via admission controllers

#### Kubernetes Network Security

*   designed to be backward compatible with VMs or physical hosts
*   Every pod can talk to other pods
*   Pod networking implemented via networking plugins
*   Network policies let you control inter-pod traffic
*   Network plugin must support network policies
*   Use the **podSelector** and **matchLabels** statements to match the pods to apply the network policy 

#### Secrets Management

*   Allow pods to get access to secrets
*   Kubernetes components need access to secrets
*   Secret object: key-value pair, stored in etcd as base64 endoded

### Securing Kubernetes Cluster

Five Key goals to secure a Kubernetes cluster:

#### Protect API Server traffic

*   Client must connect via secure port, then authenticate
*   Ensure API server pod specification file **kube-apiserver.yaml** is set correctly
    *   **\--insecure-port** is set to 0 
    *   **\--secure-port** is between 1 and 65535
    *   **\--insecure-bind-address** does not exist
    *   **\--client-ca-file** is set to correct path
    *   **\--tls-cert-file** is set to correct path
    *   **\--tls-private-key-file** is set to correct path
*   API server should be configured to accept and serve only HTTPS traffic

#### Protect cluster components

*   Protect etcd by restrict access to only serve clients over TLS
*   Disable anonymous requests in Kubelet

#### Protect resource usage

*   Protect from noisy neighbor that wants to hog resources
    *   help to prevent denial of service
*   Use ResourceQuota to prevent pods that don't meet the pre-specified CPU and memory limits from running
*   Use LimitRange to prevent requesting high values of resources

#### Protect network

*   Implement namespaces
*   Utilize network policies
*   Control access to load balancers

#### Protect secrets

*   Encrypt secrets at rest
    *   Latest K8s version support encryption in ectd
*   Protect secrets in transit
*   Rotate your secrets frequently
*   Remove the Service Account token when it is no longer needed
*   Shorten the validity timespan of a secret