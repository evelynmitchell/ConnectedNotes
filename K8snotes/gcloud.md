gcloud

Command name argument expected.
```
gcloud GROUP | COMMAND [--account=ACCOUNT]
        [--billing-project=BILLING_PROJECT] [--configuration=CONFIGURATION]
        [--flags-file=YAML_FILE] [--flatten=[KEY,...]] [--format=FORMAT]
        [--help] [--project=PROJECT_ID] [--quiet, -q]
        [--verbosity=VERBOSITY; default="warning"] [--version, -v] [-h]
        [--impersonate-service-account=SERVICE_ACCOUNT_EMAIL] [--log-http]
        [--trace-token=TRACE_TOKEN] [--no-user-output-enabled]

GROUP is to limit the scope of the command to a related set of tasks, permissions or resources. Examples of groups include auth: to manage oauth2 tokens, container: to manage container resources, kms to manage keys. Most uses of cloud will include a Group.

COMMAND are rarely used. init on initial setup, and info to get information about the gcloud setup. 

You will have to run gcloud init on every new session. There may be a way to save the configuration.
   gcloud topic configurations
      collections of key-value pairs
        default zone, verbosity, project id, active user or service account.
             gcloud config configurations create my-config
      find the path to the active config 
         gcloud info --format="get(config.paths.active_config_path)"
         gcloud config list
         gcloud config set
         
gcloud --help` to see the Cloud Platform services you can interact with. And run `gcloud help COMMAND` to get help on any gcloud command.
Run `gcloud topic --help` to learn about advanced features of the SDK like arg files and output formatting



```
Available command groups for gcloud:

  AI and Machine Learning
      ai-platform             Manage AI Platform jobs and models.
      ml                      Use Google Cloud machine learning capabilities.
      ml-engine               Manage AI Platform jobs and models.

  API Platform and Ecosystems
      endpoints               Create, enable and manage API services.
      recommender             Manage Cloud recommendations and recommendation
                              rules.
      services                List, enable and disable APIs and services.

  Anthos CLI
      anthos                  Anthos command Group.

  CI/CD
      scheduler               Manage Cloud Scheduler jobs and schedules.
      tasks                   Manage Cloud Tasks queues and tasks.

  Compute
      app                     Manage your App Engine deployments.
      compute                 Create and manipulate Compute Engine resources.
      container               Deploy and manage clusters of machines for running
                              containers.
      functions               Manage Google Cloud Functions.
      run                     Manage your Cloud Run applications.

  Data Analytics
      composer                Create and manage Cloud Composer Environments.
      data-catalog            Manage Cloud Data Catalog resources.
      dataflow                Manage Google Cloud Dataflow resources.
      dataproc                Create and manage Google Cloud Dataproc clusters
                              and jobs.
      pubsub                  Manage Cloud Pub/Sub topics, subscriptions, and
                              snapshots.

  Databases
      bigtable                Manage your Cloud Bigtable storage.
      datastore               Manage your Cloud Datastore resources.
      firestore               Manage your Cloud Firestore resources.
      spanner                 Command groups for Cloud Spanner.
      sql                     Create and manage Google Cloud SQL databases.

  Identity
      active-directory        Manage Managed Microsoft AD resources.

  Identity and Security
      access-context-manager  Manage Access Context Manager resources.
      auth                    Manage oauth2 credentials for the Google Cloud
                              SDK.
      iam                     Manage IAM service accounts and keys.
      iap                     Manage IAP policies.
      kms                     Manage cryptographic keys in the cloud.
      policy-troubleshoot     Troubleshoot Google Cloud Platform policies.
      resource-manager        Manage Cloud Resources.
      secrets                 Manage secrets on Google Cloud.Pick cloud project to use: 
 [1] firstproject-255323
 [2] k8splayground-277617
 [3] make4covid
 [4] math-playground-181015
 [5] Create a new project

  Management Tools
      builds                  Create and manage builds for Google Cloud Build.
      debug                   Commands for interacting with the Cloud Debugger.
      deployment-manager      Manage deployments of cloud resources.
      logging                 Manage Stackdriver Logging.
      organizations           Create and manage Google Cloud Platform
                              Organizations.
      projects                Create and manage project access policies.

  Mobile
      firebase                Work with Google Firebase.

  Monitoring
      monitoring              Manage Cloud Monitoring dashboards.

  Networking
      dns                     Manage your Cloud DNS managed-zones and
                              record-sets.
      domains                 Manage domains for your Google Cloud projects.
      network-management      Manage Network Management resources.

  SDK Tools
      alpha                   Alpha versions of gcloud commands.
      beta                    Beta versions of gcloud commands.
      components              List, install, update, or remove Google Cloud SDK
                              components.
      config                  View and edit Cloud SDK properties.
      meta                    Cloud meta introspection commands.
      source                  Cloud git repository commands.
      topic                   gcloud supplementary help.

  Security
      asset                   Manage the Cloud Asset Inventory.

  Solutions
      healthcare              Manage Cloud Healthcare resources.

  Storage
      filestore               Create and manipulate Cloud Filestore resources.
      redis                   Manage Cloud Memorystore Redis resources.

Available commands for gcloud:

  Other
      docker                  *(DEPRECATED)*  Enable Docker CLI access to Google
                              Container Registry.
      survey                  Invoke a customer satisfaction survey for Cloud
                              SDK.

  SDK Tools
      feedback                Provide feedback to the Google Cloud SDK team.
      help                    Search gcloud help text.
      info                    Display information about the current gcloud
                              environment.
      init                    Initialize or reinitialize gcloud.
      version                 Print version information for Cloud SDK
                              components.

## Cheat sheet

```
gcloud cheat-sheet
  Getting started
    Get going with the gcloud command-line tool

      ▪ gcloud init: Initialize, authorize, and configure the gcloud tool.
      ▪ gcloud version: Display version and installed components.
      ▪ gcloud components install: Install specific components.
      ▪ gcloud components update: Update your Cloud SDK to the latest
        version.
      ▪ gcloud config set project: Set a default Google Cloud project to work
        on.
      ▪ gcloud info: Display current gcloud tool environment details.
```

It's possible to have multiple accounts set up and switch between them, in gcloud init, by doing the url authentication dance from a different logged in browser.

```
gcloud auth list

gcloud config set account `ACCOUNT`
```

### Container and cluster management in gcloud

```
  Docker & Google Kubernetes Engine (GKE)
    Manage containerized applications on Kubernetes

      ▪ gcloud auth configure-docker: Register the gcloud tool as a Docker
        credential helper.
      ▪ gcloud container clusters create: Create a cluster to run GKE
        containers.
      ▪ gcloud container clusters list: List clusters for running GKE
        containers.
      ▪ gcloud container clusters get-credentials: Update kubeconfig to get
        kubectl to use a GKE cluster.
      ▪ gcloud container images list-tags: List tag and digest metadata for a
        container image.

```

Interesting that gcloud can be set up as a docker credentials helper (a local store of credentials which are passed into a container).

```
gcloud container clusters list
```
lists clusters as expected.
NAME              LOCATION    MASTER_VERSION  MASTER_IP       MACHINE_TYPE   NODE_VERSION    NUM_NODES  STATUS

```
gcloud container clusters get-credentials nginx102-cluster  --zone us-west1-c
```

### list images

```
gcloud container images list
Listed 0 items.
Only listing images in gcr.io/k8splayground-277617. Use --repository to list images in other repositories.
efm@efm:~/.kube$ gcloud container images list --repository
ERROR: (gcloud.container.images.list) argument --repository: expected one argument
Usage: gcloud container images list [optional flags]
  optional flags may be  --filter | --help | --limit | --page-size |
                         --repository | --sort-by | --uri

For detailed information on this command and its flags, run:
  gcloud container images list --help
```
There aren't any images. Which means we have no image repo set up.

## VMs and compute

```
gcloud compute instances list
NAME                                             ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
gke-networklb-default-pool-fa378575-54k3         us-west1-b  n1-standard-1               10.138.0.13  xx  RUNNING
gke-networklb-default-pool-fa378575-5rjb         us-west1-b  n1-standard-1               10.138.0.14  xx4    RUNNING
gke-networklb-default-pool-fa378575-c6mc         us-west1-b  n1-standard-1               10.138.0.12  3xx    RUNNING
gke-nginx102-cluster-default-pool-7e39b435-4bw4  us-west1-c  n1-standard-1               10.138.0.40  3xx6.29    RUNNING
gke-nginx102-cluster-default-pool-7e39b435-hx0z  us-west1-c  n1-standard-1               10.138.0.39  xx1   RUNNING
gke-nginx102-cluster-default-pool-7e39b435-pmq3  us-west1-c  n1-standard-1               10.138.0.41  3x3  RUNNING
```

### SSH into VM

```
gcloud compute ssh root@gke-nginx102-cluster-default-pool-7e39b435-4bw4 --zone us-west1-c
```

## Logging

### List the logs
```
gcloud logging logs list
projects/k8splayground-277617/logs/cloudaudit.googleapis.com%2Factivity
projects/k8splayground-277617/logs/cloudaudit.googleapis.com%2Fsystem_event
projects/k8splayground-277617/logs/compute.googleapis.com%2Factivity_log
projects/k8splayground-277617/logs/compute.googleapis.com%2Fshielded_vm_integrity
projects/k8splayground-277617/logs/container-runtime
projects/k8splayground-277617/logs/docker
projects/k8splayground-277617/logs/events
projects/k8splayground-277617/logs/kube-container-runtime-monitor
projects/k8splayground-277617/logs/kube-node-configuration
projects/k8splayground-277617/logs/kube-node-installation
projects/k8splayground-277617/logs/kube-proxy
projects/k8splayground-277617/logs/kubelet
projects/k8splayground-277617/logs/kubelet-monitor
projects/k8splayground-277617/logs/node-problem-detector
projects/k8splayground-277617/logs/requests
projects/k8splayground-277617/logs/stderr
projects/k8splayground-277617/logs/stdout
```

### Read the logs



```
gcloud logging read projects/k8splayground-277617/logs/stdout
---
insertId: 86lib8lk0py4itwow
jsonPayload:
  data:
    command: forward-worker
    remote: 10.16.2.44:59640
    session: 23056.4.1.102
    worker-address: 10.16.2.43:39149
    worker-platform: linux
    worker-tags: ''
  level: info
  message: tsa.connection.channel.command.heartbeat.done
  source: tsa
  timestamp: '2020-07-14T20:29:47.404276778Z'
labels:
  k8s-pod/app: concourse-1594669118-web
  k8s-pod/pod-template-hash: 59cf68db98
  k8s-pod/release: concourse-1594669118
logName: projects/k8splayground-277617/logs/stdout
receiveTimestamp: '2020-07-14T20:29:53.583099684Z'
resource:
  labels:
    cluster_name: nginx102-cluster
    container_name: concourse-1594669118-web
    location: us-west1-c
    namespace_name: default
    pod_name: concourse-1594669118-web-59cf68db98-rhl4c
    project_id: k8splayground-277617
  type: k8s_container
severity: INFO
timestamp: '2020-07-14T20:29:47.404536087Z'
...
```
### ACL

https://cloud.google.com/storage/docs/access-control/lists

# TODO
[ ] Set up k8splayground image repository





