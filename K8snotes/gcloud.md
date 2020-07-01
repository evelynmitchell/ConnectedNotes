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
      secrets                 Manage secrets on Google Cloud.

  Internet of Things
      iot                     Manage Cloud IoT resources.

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

                          



