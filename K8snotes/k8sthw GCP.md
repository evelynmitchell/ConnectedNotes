k8sthw GCP

Created two Ubuntu 20.04 VMs
Forked https://github.com/evelynmitchell/kubernetes-the-hard-way
Installed gcloud 
   # sudo snap install google-cloud-sdk --classic
The snap needs extra priviledges, thus --classic

```
  # gcloud init

Welcome! This command will take you through the configuration of gcloud.

Your current configuration has been set to: [default]

You can skip diagnostics next time by using the following flag:
  gcloud init --skip-diagnostics

Network diagnostic detects and fixes local network connection issues.
Checking network connection...done.                                                                                                                          
Reachability Check passed.
Network diagnostic passed (1/1 checks passed).

You must log in to continue. Would you like to log in (Y/n)?  y

Your browser has been opened to visit:

<a long sensitive url>
```

When I visit this, it asks me to grant permission to gcloud to manipulate my gcp resources.

```

QStandardPaths: XDG_RUNTIME_DIR points to non-existing path '/run/user/1000/snap.google-cloud-sdk', please create it with 0700 permissions.
You are logged in as: [efmphone@gmail.com].

Pick cloud project to use: 
 [1] <project>
 [2] <project>
 [3] <project>
 [4] k8sthw-280616
 [5] <project>
 [6] <project>
 [7] <project>
 [8] <project>
 [9] <project>
 [10] Create a new project
Please enter numeric choice or text value (must exactly match list item):
4
Your current project has been set to: [k8sthw-280616].

Do you want to configure a default Compute Region and Zone? (Y/n)? y
Please enter a value between 1 and 74, or a value present in the list:  10

Your project default Compute Engine zone has been set to [us-central1-b].
You can change it by running [gcloud config set compute/zone NAME].

Your project default Compute Engine region has been set to [us-central1].
You can change it by running [gcloud config set compute/region NAME].

Created a default .boto configuration file at [/home/efm/.boto]. See this file and
[https://cloud.google.com/storage/docs/gsutil/commands/config] for more
information about configuring Google Cloud Storage.
Your Google Cloud SDK is configured and ready to use!

* Commands that require authentication will use efmphone@gmail.com by default
* Commands will reference project `k8sthw-280616` by default
* Compute Engine commands will use region `us-central1` by default
* Compute Engine commands will use zone `us-central1-b` by default

Run `gcloud help config` to learn how to change individual settings

This gcloud configuration is called [default]. You can create additional configurations if you work with multiple accounts and/or projects.
Run `gcloud topic configurations` to learn more.

Some things to try next:

* Run `gcloud --help` to see the Cloud Platform services you can interact with. And run `gcloud help COMMAND` to get help on any gcloud command.
* Run `gcloud topic --help` to learn about advanced features of the SDK like arg files and output formatting
```

Then authorize

```
  # gcloud auth login
```

Grant permissions again

I didn't do 
gcloud config set compute/region us-west1
because I had already set the region.



