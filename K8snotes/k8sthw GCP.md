k8sthw GCP

Created two Ubuntu 20.04 VMs
Forked https://github.com/evelynmitchell/kubernetes-the-hard-way
Installed gcloud 

```
   # sudo snap install google-cloud-sdk --classic
```
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

## Install the rest of the client tools

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssljson

wget -q --show-progress --https-only --timestamping \
>   https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssl \
>   https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssljson
cfssl                                   100%[=============================================================================>]  19.62M  4.79MB/s    in 4.1s    
cfssljson                               100%[=============================================================================>]  12.08M  2.68MB/s    in 4.5s    
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ chmod +x cfssl cfssljson

```

I hope this doesn't break anything

```
sudo mv cfssl cfssljson /usr/local/bin/
```

```
cfssl version
Version: 1.3.4
Revision: dev
Runtime: go1.13

cfssljson --version
Version: 1.3.4
Revision: dev
Runtime: go1.13

```

I'm going  to use a snap for kubectl

```
kubectl

Command 'kubectl' not found, but can be installed with:

sudo snap install kubectl 

sudo snap install kubectl --classic
kubectl 1.18.4 from Canonical✓ installed

kubectl version --client
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.4", GitCommit:"c96aede7b5205121079932896c4ad89bb93260af", GitTreeState:"clean", BuildDate:"2020-06-18T17:02:08Z", GoVersion:"go1.13.12", Compiler:"gc", Platform:"linux/amd64"}

```

## Network setup

```
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/networks/kubernetes-the-hard-way].
NAME                     SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
kubernetes-the-hard-way  CUSTOM       REGIONAL

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network kubernetes-the-hard-way --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network kubernetes-the-hard-way --allow tcp:22,tcp:3389,icmp

gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24

gcloud compute networks subnets create kubernetes \
>   --network kubernetes-the-hard-way \
>   --range 10.240.0.0/24
Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/subnetworks/kubernetes].
NAME        REGION       NETWORK                  RANGE
kubernetes  us-central1  kubernetes-the-hard-way  10.240.0.0/24

```

### Firewall rules

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16


gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
>   --allow tcp,udp,icmp \
>   --network kubernetes-the-hard-way \
>   --source-ranges 10.240.0.0/24,10.200.0.0/16
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-internal].         
Creating firewall...done.                                                                                                                                    
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW         DENY  DISABLED
kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp        False

gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0

  Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-external].         
Creating firewall...done.                                                                                                                                    
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False

gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"

gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False
kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp                False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

```

### Static public IP

```
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)

  Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/addresses/kubernetes-the-hard-way].
  ```

Verify

```
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
NAME                     ADDRESS/RANGE  TYPE      PURPOSE  NETWORK  REGION       SUBNET  STATUS
kubernetes-the-hard-way  34.72.53.87    EXTERNAL                    us-central1          RESERVED

```

### Create compute instances

```
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
done

NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [controller-0]: https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/operations/operation-1593271489609-5a9126d2b3dd0-cc68cc0f-8d9f72fc
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [controller-1]: https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/operations/operation-1593271492295-5a9126d543abc-fa7daf5a-d1c34b36
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [controller-2]: https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/operations/operation-1593271494846-5a9126d7b2766-ae30ca2a-9ef9a5b2
Use [gcloud compute operations describe URI] command to check the status of the operation(s).

```

### Create worker nodes

```
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done

NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [worker-0]: https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/operations/operation-1593271646793-5a9127689aeb7-cd2994d4-79d9e505
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [worker-1]: https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/operations/operation-1593271649386-5a91276b141d1-94de3e47-6abe69fa
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
NOTE: The users will be charged for public IPs when VMs are created.
Instance creation in progress for [worker-2]: https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/operations/operation-1593271651887-5a91276d76a25-1b44cddc-ade9a142
Use [gcloud compute operations describe URI] command to check the status of the operation(s).
```

### Verify instances

```
gcloud compute instances list

NAME          ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP      STATUS
controller-0  us-central1-b  n1-standard-1               10.240.0.10  35.226.68.16     RUNNING
controller-1  us-central1-b  n1-standard-1               10.240.0.11  130.211.222.139  RUNNING
controller-2  us-central1-b  n1-standard-1               10.240.0.12  35.223.99.40     RUNNING
worker-0      us-central1-b  n1-standard-1               10.240.0.20  34.70.114.213    RUNNING
worker-1      us-central1-b  n1-standard-1               10.240.0.21  35.226.118.223   RUNNING
worker-2      us-central1-b  n1-standard-1               10.240.0.22  35.232.160.221   RUNNING
```

### Configure SSH access

```
gcloud compute ssh controller-0
```

There was a key creation and password setup which I will not include here

```
gcloud compute ssh controller-0
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-1029-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Jun 27 15:34:34 UTC 2020

  System load:  0.03               Processes:           95
  Usage of /:   0.7% of 193.66GB   Users logged in:     0
  Memory usage: 5%                 IP address for ens4: 10.240.0.10
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


efm@controller-0:~$ 
```

### I'm In

```
exit
logout
Connection to 35.226.68.16 closed.
```


### Provisioning CA

"In this lab you will provision a PKI Infrastructure using CloudFlare's PKI toolkit, cfssl, then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy."


Note that the { } here are a grouping in bash, not json.
The reason using {} is so that the changes are executed within the original shell context, rather than creating a subshell which would not share the same enviornment variables.


```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

Yup it worked

```
2020/06/27 09:45:41 [INFO] generating a new CA key and certificate from CSR
2020/06/27 09:45:41 [INFO] generate received request
2020/06/27 09:45:41 [INFO] received CSR
2020/06/27 09:45:41 [INFO] generating key: rsa-2048
2020/06/27 09:45:41 [INFO] encoded CSR
2020/06/27 09:45:41 [INFO] signed certificate with serial number 721286613313691851076167236053048368045875461963

```

Check for the files

```
ls 
ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  CONTRIBUTING.md  COPYRIGHT.md  deployments  docs  LICENSE  README.md
```

Generate admin certs  

```
{

cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}

2020/06/27 09:48:15 [INFO] generate received request
2020/06/27 09:48:15 [INFO] received CSR
2020/06/27 09:48:15 [INFO] generating key: rsa-2048
2020/06/27 09:48:15 [INFO] encoded CSR
2020/06/27 09:48:15 [INFO] signed certificate with serial number 261229676193254128938752896688081251499488258633
2020/06/27 09:48:15 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

ls 
admin.csr       admin-key.pem  ca-config.json  ca-csr.json  ca.pem           COPYRIGHT.md  docs     README.md
admin-csr.json  admin.pem      ca.csr          ca-key.pem   CONTRIBUTING.md  deployments   LICENSE

```

### kubelet client certificates

```
for instance in worker-0 worker-1 worker-2; do
cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

EXTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].networkIP)')

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done

2020/06/27 09:52:11 [INFO] generate received request
2020/06/27 09:52:11 [INFO] received CSR
2020/06/27 09:52:11 [INFO] generating key: rsa-2048
2020/06/27 09:52:11 [INFO] encoded CSR
2020/06/27 09:52:11 [INFO] signed certificate with serial number 332524972477543552870317305068831231870986627701
2020/06/27 09:52:14 [INFO] generate received request
2020/06/27 09:52:14 [INFO] received CSR
2020/06/27 09:52:14 [INFO] generating key: rsa-2048
2020/06/27 09:52:14 [INFO] encoded CSR
2020/06/27 09:52:14 [INFO] signed certificate with serial number 212964972211849591602119478588852920830888086269
2020/06/27 09:52:17 [INFO] generate received request
2020/06/27 09:52:17 [INFO] received CSR
2020/06/27 09:52:17 [INFO] generating key: rsa-2048
2020/06/27 09:52:18 [INFO] encoded CSR
2020/06/27 09:52:18 [INFO] signed certificate with serial number 56738396642278894067004804745041309224421289106

ls 
admin.csr       admin.pem       ca-csr.json  CONTRIBUTING.md  docs       worker-0.csr       worker-0.pem       worker-1-key.pem  worker-2-csr.json
admin-csr.json  ca-config.json  ca-key.pem   COPYRIGHT.md     LICENSE    worker-0-csr.json  worker-1.csr       worker-1.pem      worker-2-key.pem
admin-key.pem   ca.csr          ca.pem       deployments      README.md  worker-0-key.pem   worker-1-csr.json  worker-2.csr      worker-2.pem

```


### Controller manager certificate

```
{

cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}

2020/06/27 09:54:46 [INFO] generate received request
2020/06/27 09:54:46 [INFO] received CSR
2020/06/27 09:54:46 [INFO] generating key: rsa-2048
2020/06/27 09:54:46 [INFO] encoded CSR
2020/06/27 09:54:46 [INFO] signed certificate with serial number 154346400804599157115195901801002368810160151040
2020/06/27 09:54:46 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

ls 
admin.csr       ca.csr           COPYRIGHT.md                      kube-controller-manager-key.pem  worker-0-csr.json  worker-1-key.pem   worker-2.pem
admin-csr.json  ca-csr.json      deployments                       kube-controller-manager.pem      worker-0-key.pem   worker-1.pem
admin-key.pem   ca-key.pem       docs                              LICENSE                          worker-0.pem       worker-2.csr
admin.pem       ca.pem           kube-controller-manager.csr       README.md                        worker-1.csr       worker-2-csr.json
ca-config.json  CONTRIBUTING.md  kube-controller-manager-csr.json  worker-0.csr                     worker-1-csr.json  worker-2-key.pem
```

### Kube Proxy Client Certificate

```
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}

2020/06/27 09:57:24 [INFO] generate received request
2020/06/27 09:57:24 [INFO] received CSR
2020/06/27 09:57:24 [INFO] generating key: rsa-2048
2020/06/27 09:57:24 [INFO] encoded CSR
2020/06/27 09:57:24 [INFO] signed certificate with serial number 279407610448290674768414334222698037901375872992
2020/06/27 09:57:24 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

ls 
admin.csr       ca.csr           COPYRIGHT.md                      kube-controller-manager-key.pem  kube-proxy.pem     worker-0-key.pem   worker-1.pem
admin-csr.json  ca-csr.json      deployments                       kube-controller-manager.pem      LICENSE            worker-0.pem       worker-2.csr
admin-key.pem   ca-key.pem       docs                              kube-proxy.csr                   README.md          worker-1.csr       worker-2-csr.json
admin.pem       ca.pem           kube-controller-manager.csr       kube-proxy-csr.json              worker-0.csr       worker-1-csr.json  worker-2-key.pem
ca-config.json  CONTRIBUTING.md  kube-controller-manager-csr.json  kube-proxy-key.pem               worker-0-csr.json  worker-1-key.pem   worker-2.pem
``` 

### Scheduler Client Certificate

```
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}

2020/06/27 09:59:31 [INFO] generate received request
2020/06/27 09:59:31 [INFO] received CSR
2020/06/27 09:59:31 [INFO] generating key: rsa-2048
2020/06/27 09:59:31 [INFO] encoded CSR
2020/06/27 09:59:31 [INFO] signed certificate with serial number 58030601383547395617318372447276734782671195971
2020/06/27 09:59:31 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").


ls 
admin.csr       ca-csr.json      docs                              kube-proxy-csr.json      kube-scheduler.pem  worker-0.pem       worker-2-csr.json
admin-csr.json  ca-key.pem       kube-controller-manager.csr       kube-proxy-key.pem       LICENSE             worker-1.csr       worker-2-key.pem
admin-key.pem   ca.pem           kube-controller-manager-csr.json  kube-proxy.pem           README.md           worker-1-csr.json  worker-2.pem
admin.pem       CONTRIBUTING.md  kube-controller-manager-key.pem   kube-scheduler.csr       worker-0.csr        worker-1-key.pem
ca-config.json  COPYRIGHT.md     kube-controller-manager.pem       kube-scheduler-csr.json  worker-0-csr.json   worker-1.pem
ca.csr          deployments      kube-proxy.csr                    kube-scheduler-key.pem   worker-0-key.pem    worker-2.csr

```

### Kubernetes Server API certificate

```
{

KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,${KUBERNETES_HOSTNAMES} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}

2020/06/27 10:01:06 [INFO] generate received request
2020/06/27 10:01:06 [INFO] received CSR
2020/06/27 10:01:06 [INFO] generating key: rsa-2048
2020/06/27 10:01:06 [INFO] encoded CSR
2020/06/27 10:01:06 [INFO] signed certificate with serial number 411201534192533111284304824528746495470220956073

ls 
admin.csr       ca-key.pem                   kube-controller-manager-csr.json  kubernetes.csr           kube-scheduler.pem  worker-1.csr       worker-2.pem
admin-csr.json  ca.pem                       kube-controller-manager-key.pem   kubernetes-csr.json      LICENSE             worker-1-csr.json
admin-key.pem   CONTRIBUTING.md              kube-controller-manager.pem       kubernetes-key.pem       README.md           worker-1-key.pem
admin.pem       COPYRIGHT.md                 kube-proxy.csr                    kubernetes.pem           worker-0.csr        worker-1.pem
ca-config.json  deployments                  kube-proxy-csr.json               kube-scheduler.csr       worker-0-csr.json   worker-2.csr
ca.csr          docs                         kube-proxy-key.pem                kube-scheduler-csr.json  worker-0-key.pem    worker-2-csr.json
ca-csr.json     kube-controller-manager.csr  kube-proxy.pem                    kube-scheduler-key.pem   worker-0.pem        worker-2-key.pem
```

### Service account keypair

```
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}

2020/06/27 10:02:49 [INFO] generate received request
2020/06/27 10:02:49 [INFO] received CSR
2020/06/27 10:02:49 [INFO] generating key: rsa-2048
2020/06/27 10:02:49 [INFO] encoded CSR
2020/06/27 10:02:49 [INFO] signed certificate with serial number 383432656597844388981569592219872688003981747416
2020/06/27 10:02:49 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

ls 
admin.csr       ca.pem                            kube-controller-manager.pem  kubernetes.pem           service-account-csr.json  worker-1-csr.json
admin-csr.json  CONTRIBUTING.md                   kube-proxy.csr               kube-scheduler.csr       service-account-key.pem   worker-1-key.pem
admin-key.pem   COPYRIGHT.md                      kube-proxy-csr.json          kube-scheduler-csr.json  service-account.pem       worker-1.pem
admin.pem       deployments                       kube-proxy-key.pem           kube-scheduler-key.pem   worker-0.csr              worker-2.csr
ca-config.json  docs                              kube-proxy.pem               kube-scheduler.pem       worker-0-csr.json         worker-2-csr.json
ca.csr          kube-controller-manager.csr       kubernetes.csr               LICENSE                  worker-0-key.pem          worker-2-key.pem
ca-csr.json     kube-controller-manager-csr.json  kubernetes-csr.json          README.md                worker-0.pem              worker-2.pem
ca-key.pem      kube-controller-manager-key.pem   kubernetes-key.pem           service-account.csr      worker-1.csr

``` 

### Distribute certificates

```
for instance in worker-0 worker-1 worker-2; do   gcloud compute scp ca.pem ${instance}-key.pem ${instance}.pem ${instance}:~/; done
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
ca.pem                                                                                                                      100% 1318    29.4KB/s   00:00    
worker-0-key.pem                                                                                                            100% 1679    40.7KB/s   00:00    
worker-0.pem                                                                                                                100% 1493    40.8KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
ca.pem                                                                                                                      100% 1318    37.2KB/s   00:00    
worker-1-key.pem                                                                                                            100% 1675    39.0KB/s   00:00    
worker-1.pem                                                                                                                100% 1493    37.8KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
ca.pem                                                                                                                      100% 1318    34.7KB/s   00:00    
worker-2-key.pem                                                                                                            100% 1675    38.2KB/s   00:00    
worker-2.pem      
```

```
for instance in controller-0 controller-1 controller-2; do
>   gcloud compute scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
>     service-account-key.pem service-account.pem ${instance}:~/
> done
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
ca.pem                                                                                                                      100% 1318    37.0KB/s   00:00    
ca-key.pem                                                                                                                  100% 1675    41.2KB/s   00:00    
kubernetes-key.pem                                                                                                          100% 1679    30.8KB/s   00:00    
kubernetes.pem                                                                                                              100% 1663    40.6KB/s   00:00    
service-account-key.pem                                                                                                     100% 1675    42.7KB/s   00:00    
service-account.pem                                                                                                         100% 1440    33.8KB/s   00:00    
Warning: Permanently added 'compute.4320877203762633259' (ECDSA) to the list of known hosts.
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
ca.pem                                                                                                                      100% 1318    33.5KB/s   00:00    
ca-key.pem                                                                                                                  100% 1675    41.7KB/s   00:00    
kubernetes-key.pem                                                                                                          100% 1679    46.5KB/s   00:00    
kubernetes.pem                                                                                                              100% 1663    46.3KB/s   00:00    
service-account-key.pem                                                                                                     100% 1675    42.3KB/s   00:00    
service-account.pem                                                                                                         100% 1440    39.9KB/s   00:00    
Warning: Permanently added 'compute.4767168148843190824' (ECDSA) to the list of known hosts.
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
ca.pem                                                                                                                      100% 1318    33.0KB/s   00:00    
ca-key.pem                                                                                                                  100% 1675    43.0KB/s   00:00    
kubernetes-key.pem                                                                                                          100% 1679    44.6KB/s   00:00    
kubernetes.pem                                                                                                              100% 1663    45.7KB/s   00:00    
service-account-key.pem                                                                                                     100% 1675    45.0KB/s   00:00    
service-account.pem                                                                                                         100% 1440    40.6KB/s   00:00 
```

### Generating Kubernetes Configuration Files for Authentication

```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

set | more 
KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local
KUBERNETES_PUBLIC_ADDRESS=34.72.53.87
```

The following commands must be run in the same directory used to generate the SSL certificates during the Generating TLS Certificates lab

```
for instance in worker-0 worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done

Cluster "kubernetes-the-hard-way" set.
User "system:node:worker-0" set.
Context "default" created.
Switched to context "default".
Cluster "kubernetes-the-hard-way" set.
User "system:node:worker-1" set.
Context "default" created.
Switched to context "default".
Cluster "kubernetes-the-hard-way" set.
User "system:node:worker-2" set.
Context "default" created.
Switched to context "default".

ls 
admin.csr       CONTRIBUTING.md                   kube-proxy-csr.json      kube-scheduler-key.pem    worker-0-csr.json    worker-2.csr
admin-csr.json  COPYRIGHT.md                      kube-proxy-key.pem       kube-scheduler.pem        worker-0-key.pem     worker-2-csr.json
admin-key.pem   deployments                       kube-proxy.pem           LICENSE                   worker-0.kubeconfig  worker-2-key.pem
admin.pem       docs                              kubernetes.csr           README.md                 worker-0.pem         worker-2.kubeconfig
ca-config.json  kube-controller-manager.csr       kubernetes-csr.json      service-account.csr       worker-1.csr         worker-2.pem
ca.csr          kube-controller-manager-csr.json  kubernetes-key.pem       service-account-csr.json  worker-1-csr.json
ca-csr.json     kube-controller-manager-key.pem   kubernetes.pem           service-account-key.pem   worker-1-key.pem
ca-key.pem      kube-controller-manager.pem       kube-scheduler.csr       service-account.pem       worker-1.kubeconfig
ca.pem          kube-proxy.csr                    kube-scheduler-csr.json  worker-0.csr              worker-1.pem

```


###  kube-proxy Kubernetes Configuration File

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}

Cluster "kubernetes-the-hard-way" set.
User "system:kube-proxy" set.
Context "default" created.
Switched to context "default".
   
ls 
admin.csr       CONTRIBUTING.md                   kube-proxy-csr.json    kube-scheduler-csr.json   worker-0.csr         worker-1.pem
admin-csr.json  COPYRIGHT.md                      kube-proxy-key.pem     kube-scheduler-key.pem    worker-0-csr.json    worker-2.csr
admin-key.pem   deployments                       kube-proxy.kubeconfig  kube-scheduler.pem        worker-0-key.pem     worker-2-csr.json
admin.pem       docs                              kube-proxy.pem         LICENSE                   worker-0.kubeconfig  worker-2-key.pem
ca-config.json  kube-controller-manager.csr       kubernetes.csr         README.md                 worker-0.pem         worker-2.kubeconfig
ca.csr          kube-controller-manager-csr.json  kubernetes-csr.json    service-account.csr       worker-1.csr         worker-2.pem
ca-csr.json     kube-controller-manager-key.pem   kubernetes-key.pem     service-account-csr.json  worker-1-csr.json
ca-key.pem      kube-controller-manager.pem       kubernetes.pem         service-account-key.pem   worker-1-key.pem
ca.pem          kube-proxy.csr                    kube-scheduler.csr     service-account.pem       worker-1.kubeconfig
```

###  kube-controller-manager Kubernetes Configuration File

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}

Cluster "kubernetes-the-hard-way" set.
User "system:kube-controller-manager" set.
Context "default" created.
Switched to context "default".

ls 
admin.csr       CONTRIBUTING.md                     kube-proxy.csr         kube-scheduler.csr        service-account.pem  worker-1.kubeconfig
admin-csr.json  COPYRIGHT.md                        kube-proxy-csr.json    kube-scheduler-csr.json   worker-0.csr         worker-1.pem
admin-key.pem   deployments                         kube-proxy-key.pem     kube-scheduler-key.pem    worker-0-csr.json    worker-2.csr
admin.pem       docs                                kube-proxy.kubeconfig  kube-scheduler.pem        worker-0-key.pem     worker-2-csr.json
ca-config.json  kube-controller-manager.csr         kube-proxy.pem         LICENSE                   worker-0.kubeconfig  worker-2-key.pem
ca.csr          kube-controller-manager-csr.json    kubernetes.csr         README.md                 worker-0.pem         worker-2.kubeconfig
ca-csr.json     kube-controller-manager-key.pem     kubernetes-csr.json    service-account.csr       worker-1.csr         worker-2.pem
ca-key.pem      kube-controller-manager.kubeconfig  kubernetes-key.pem     service-account-csr.json  worker-1-csr.json
ca.pem          kube-controller-manager.pem         kubernetes.pem         service-account-key.pem   worker-1-key.pem
```

###  kube-scheduler Kubernetes Configuration File

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}

Cluster "kubernetes-the-hard-way" set.
User "system:kube-scheduler" set.
Context "default" created.
Switched to context "default".

ls 
admin.csr       CONTRIBUTING.md                     kube-proxy.csr         kube-scheduler.csr         service-account-key.pem  worker-1-key.pem
admin-csr.json  COPYRIGHT.md                        kube-proxy-csr.json    kube-scheduler-csr.json    service-account.pem      worker-1.kubeconfig
admin-key.pem   deployments                         kube-proxy-key.pem     kube-scheduler-key.pem     worker-0.csr             worker-1.pem
admin.pem       docs                                kube-proxy.kubeconfig  kube-scheduler.kubeconfig  worker-0-csr.json        worker-2.csr
ca-config.json  kube-controller-manager.csr         kube-proxy.pem         kube-scheduler.pem         worker-0-key.pem         worker-2-csr.json
ca.csr          kube-controller-manager-csr.json    kubernetes.csr         LICENSE                    worker-0.kubeconfig      worker-2-key.pem
ca-csr.json     kube-controller-manager-key.pem     kubernetes-csr.json    README.md                  worker-0.pem             worker-2.kubeconfig
ca-key.pem      kube-controller-manager.kubeconfig  kubernetes-key.pem     service-account.csr        worker-1.csr             worker-2.pem
ca.pem          kube-controller-manager.pem         kubernetes.pem         service-account-csr.json   worker-1-csr.json
```

### admin Kubernetes Configuration File

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}

Cluster "kubernetes-the-hard-way" set.
User "admin" set.
Context "default" created.
Switched to context "default".

ls 
admin.csr         ca.pem                              kube-controller-manager.pem  kubernetes.pem             service-account-csr.json  worker-1-csr.json
admin-csr.json    CONTRIBUTING.md                     kube-proxy.csr               kube-scheduler.csr         service-account-key.pem   worker-1-key.pem
admin-key.pem     COPYRIGHT.md                        kube-proxy-csr.json          kube-scheduler-csr.json    service-account.pem       worker-1.kubeconfig
admin.kubeconfig  deployments                         kube-proxy-key.pem           kube-scheduler-key.pem     worker-0.csr              worker-1.pem
admin.pem         docs                                kube-proxy.kubeconfig        kube-scheduler.kubeconfig  worker-0-csr.json         worker-2.csr
ca-config.json    kube-controller-manager.csr         kube-proxy.pem               kube-scheduler.pem         worker-0-key.pem          worker-2-csr.json
ca.csr            kube-controller-manager-csr.json    kubernetes.csr               LICENSE                    worker-0.kubeconfig       worker-2-key.pem
ca-csr.json       kube-controller-manager-key.pem     kubernetes-csr.json          README.md                  worker-0.pem              worker-2.kubeconfig
ca-key.pem        kube-controller-manager.kubeconfig  kubernetes-key.pem           service-account.csr        worker-1.csr              worker-2.pem
```

### Distribute the config files

```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
done

for instance in worker-0 worker-1 worker-2; do
>   gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
> done
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
worker-0.kubeconfig                                                                                                         100% 6385   140.7KB/s   00:00    
kube-proxy.kubeconfig                                                                                                       100% 6323   158.8KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
worker-1.kubeconfig                                                                                                         100% 6381   139.9KB/s   00:00    
kube-proxy.kubeconfig                                                                                                       100% 6323   167.7KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
worker-2.kubeconfig                                                                                                         100% 6381   151.8KB/s   00:00    
kube-proxy.kubeconfig                                                                                                       100% 6323   168.5KB/s   00:00   
```

```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
done
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
admin.kubeconfig                                                                                                            100% 6261   148.4KB/s   00:00    
kube-controller-manager.kubeconfig                                                                                          100% 6391   126.0KB/s   00:00    
kube-scheduler.kubeconfig                                                                                                   100% 6337   158.5KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
admin.kubeconfig                                                                                                            100% 6261   151.3KB/s   00:00    
kube-controller-manager.kubeconfig                                                                                          100% 6391   165.7KB/s   00:00    
kube-scheduler.kubeconfig                                                                                                   100% 6337   156.2KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
admin.kubeconfig                                                                                                            100% 6261   145.1KB/s   00:00    
kube-controller-manager.kubeconfig                                                                                          100% 6391   141.6KB/s   00:00    
kube-scheduler.kubeconfig                                                                                                   100% 6337    68.7KB/s   00:00    
```

### Generating Data Encryption key

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to encrypt cluster data at rest.

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

### Encryption Config File

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

Interesting that those two steps are not similarly combined as the certificate generation steps were.

```
 more encryption-config.yaml 
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <sekrit=>
      - identity: {}
```

Copy the file 

```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp encryption-config.yaml ${instance}:~/
done

Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
encryption-config.yaml                                                                                                      100%  240     6.5KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
encryption-config.yaml                                                                                                      100%  240     6.3KB/s   00:00    
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
encryption-config.yaml                                                                                                      100%  240     6.7KB/s   00:00    

```

## Bootstrapping the etcd Cluster

etcd is a key value store

Kubernetes components are stateless and store cluster state in etcd. In this lab you will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

The next set of commands I will run in 3 separate shells each connecting to a controller-{0,1,2}
```
gcloud compute ssh controller-0
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-1029-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Jun 27 16:40:20 UTC 2020

  System load:  0.0                Processes:           94
  Usage of /:   0.7% of 193.66GB   Users logged in:     0
  Memory usage: 5%                 IP address for ens4: 10.240.0.10
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


Last login: Sat Jun 27 15:34:34 2020 from 66.35.36.132
efm@controller-0:~$ 

wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz"

   wget -q --show-progress --https-only --timestamping \
>   "https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz"
etcd-v3.4.0-linux-amd64.tar.gz          100%[=============================================================================>]  16.45M  46.1MB/s    in 0.4s    
efm@controller-0:~$ 

{
  tar -xvf etcd-v3.4.0-linux-amd64.tar.gz
  sudo mv etcd-v3.4.0-linux-amd64/etcd* /usr/local/bin/
}

efm@controller-0:~$ {
>   tar -xvf etcd-v3.4.0-linux-amd64.tar.gz
>   sudo mv etcd-v3.4.0-linux-amd64/etcd* /usr/local/bin/
> }
tar: Ignoring unknown extended header keyword 'SCHILY.dev'
tar: Ignoring unknown extended header keyword 'SCHILY.ino'
tar: Ignoring unknown extended header keyword 'SCHILY.nlink'
...
tar: Ignoring unknown extended header keyword 'SCHILY.nlink'
etcd-v3.4.0-linux-amd64/Documentation/metrics/v3.2.1
efm@controller-0:~$ 

```

I'm ignoring those tar header errors.

### Configure etcd

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}

INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)

ETCD_NAME=$(hostname -s)
```

Create the systemd unit file 

```
efm@controller-0:~$ cat <<EOF | sudo tee /etc/systemd/system/etcd.service
> [Unit]
> Description=etcd
> Documentation=https://github.com/coreos
> 
> [Service]
> Type=notify
> ExecStart=/usr/local/bin/etcd \\
>   --name ${ETCD_NAME} \\
>   --cert-file=/etc/etcd/kubernetes.pem \\
>   --key-file=/etc/etcd/kubernetes-key.pem \\
>   --peer-cert-file=/etc/etcd/kubernetes.pem \\
>   --peer-key-file=/etc/etcd/kubernetes-key.pem \\
>   --trusted-ca-file=/etc/etcd/ca.pem \\
>   --peer-trusted-ca-file=/etc/etcd/ca.pem \\
>   --peer-client-cert-auth \\
>   --client-cert-auth \\
>   --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
>   --listen-peer-urls https://${INTERNAL_IP}:2380 \\
>   --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
>   --advertise-client-urls https://${INTERNAL_IP}:2379 \\
>   --initial-cluster-token etcd-cluster-0 \\
>   --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \\
>   --initial-cluster-state new \\
>   --data-dir=/var/lib/etcd
> Restart=on-failure
> RestartSec=5
> 
> [Install]
> WantedBy=multi-user.target
> EOF
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name controller-0 \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://10.240.0.10:2380 \
  --listen-peer-urls https://10.240.0.10:2380 \
  --listen-client-urls https://10.240.0.10:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.240.0.10:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Start the etcd service

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
Created symlink /etc/systemd/system/multi-user.target.wants/etcd.service → /etc/systemd/system/etcd.service.
```

### Verify etcd

```
efm@controller-0:~$ sudo ETCDCTL_API=3 etcdctl member list \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false

efm@controller-1:~$ sudo ETCDCTL_API=3 etcdctl member list \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false

efm@controller-2:~$ sudo ETCDCTL_API=3 etcdctl member list \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false

```


## Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across three compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

These commands are run on each controller-{0,1,2}

```
efm@controller-0:~$ sudo mkdir -p /etc/kubernetes/config
efm@controller-0:~$ wget -q --show-progress --https-only --timestamping \
>   "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-apiserver" \
>   "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-controller-manager" \
>   "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-scheduler" \
>   "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl"
kube-apiserver                          100%[=============================================================================>] 156.90M   204MB/s    in 0.8s    
kube-controller-manager                 100%[=============================================================================>] 111.03M   165MB/s    in 0.7s    
kube-scheduler                          100%[=============================================================================>]  36.99M  60.7MB/s    in 0.6s    
kubectl                                 100%[=============================================================================>]  40.99M   162MB/s    in 0.3s    
efm@controller-0:~$ {
>   chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
>   sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
> }

{
  sudo mkdir -p /var/lib/kubernetes/

  sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem \
    encryption-config.yaml /var/lib/kubernetes/
}
efm@controller-0:~$ {
>   sudo mkdir -p /var/lib/kubernetes/
> 
>   sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
>     service-account-key.pem service-account.pem \
>     encryption-config.yaml /var/lib/kubernetes/
> }

efm@controller-0:~$ INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
>   http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)

efm@controller-0:~$ cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
> [Unit]
> Description=Kubernetes API Server
> Documentation=https://github.com/kubernetes/kubernetes
> 
> [Service]
> ExecStart=/usr/local/bin/kube-apiserver \\
>   --advertise-address=${INTERNAL_IP} \\
>   --allow-privileged=true \\
>   --apiserver-count=3 \\
>   --audit-log-maxage=30 \\
>   --audit-log-maxbackup=3 \\
>   --audit-log-maxsize=100 \\
>   --audit-log-path=/var/log/audit.log \\
>   --authorization-mode=Node,RBAC \\
>   --bind-address=0.0.0.0 \\
>   --client-ca-file=/var/lib/kubernetes/ca.pem \\
>   --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
>   --etcd-cafile=/var/lib/kubernetes/ca.pem \\
>   --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
>   --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
>   --etcd-servers=https://10.240.0.10:2379,https://10.240.0.11:2379,https://10.240.0.12:2379 \\
>   --event-ttl=1h \\
>   --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
>   --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
>   --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
>   --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
>   --kubelet-https=true \\
>   --runtime-config=api/all \\
>   --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
>   --service-cluster-ip-range=10.32.0.0/24 \\
>   --service-node-port-range=30000-32767 \\
>   --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
>   --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
>   --v=2
> Restart=on-failure
> RestartSec=5
> 
> [Install]
> WantedBy=multi-user.target
> EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=10.240.0.10 \
  --allow-privileged=true \
  --apiserver-count=3 \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/log/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \
  --etcd-servers=https://10.240.0.10:2379,https://10.240.0.11:2379,https://10.240.0.12:2379 \
  --event-ttl=1h \
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \
  --kubelet-https=true \
  --runtime-config=api/all \
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

efm@controller-0:~$ sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/

efm@controller-0:~$ cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
> [Unit]
> Description=Kubernetes Controller Manager
> Documentation=https://github.com/kubernetes/kubernetes
> 
> [Service]
> ExecStart=/usr/local/bin/kube-controller-manager \\
>   --address=0.0.0.0 \\
>   --cluster-cidr=10.200.0.0/16 \\
>   --cluster-name=kubernetes \\
>   --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
>   --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
>   --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
>   --leader-elect=true \\
>   --root-ca-file=/var/lib/kubernetes/ca.pem \\
>   --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
>   --service-cluster-ip-range=10.32.0.0/24 \\
>   --use-service-account-credentials=true \\
>   --v=2
> Restart=on-failure
> RestartSec=5
> 
> [Install]
> WantedBy=multi-user.target
> EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --address=0.0.0.0 \
  --cluster-cidr=10.200.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \
  --leader-elect=true \
  --root-ca-file=/var/lib/kubernetes/ca.pem \
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --use-service-account-credentials=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

efm@controller-0:~$ sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/

efm@controller-0:~$ cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
> apiVersion: kubescheduler.config.k8s.io/v1alpha1
> kind: KubeSchedulerConfiguration
> clientConnection:
>   kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
> leaderElection:
>   leaderElect: true
> EOF
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true

efm@controller-0:~$ cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
> [Unit]
> Description=Kubernetes Scheduler
> Documentation=https://github.com/kubernetes/kubernetes
> 
> [Service]
> ExecStart=/usr/local/bin/kube-scheduler \\
>   --config=/etc/kubernetes/config/kube-scheduler.yaml \\
>   --v=2
> Restart=on-failure
> RestartSec=5
> 
> [Install]
> WantedBy=multi-user.target
> EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --config=/etc/kubernetes/config/kube-scheduler.yaml \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

efm@controller-0:~$ {
>   sudo systemctl daemon-reload
>   sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
>   sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
> }
Created symlink /etc/systemd/system/multi-user.target.wants/kube-apiserver.service → /etc/systemd/system/kube-apiserver.service.
Created symlink /etc/systemd/system/multi-user.target.wants/kube-controller-manager.service → /etc/systemd/system/kube-controller-manager.service.
Created symlink /etc/systemd/system/multi-user.target.wants/kube-scheduler.service → /etc/systemd/system/kube-scheduler.service.

### Enable http health checks

```
efm@controller-0:~$ sudo apt-get update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]  
Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:4 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                
Get:5 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [8570 kB]                   
Get:6 http://archive.canonical.com/ubuntu bionic InRelease [10.2 kB]                                                 
Get:7 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/universe Translation-en [4941 kB]
Get:8 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [151 kB]
Get:9 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/multiverse Translation-en [108 kB]
Get:10 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [69.6 kB]
Get:11 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [1086 kB]
Get:12 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/universe Translation-en [337 kB]
Get:13 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [11.3 kB]
Get:14 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/multiverse Translation-en [4804 B]
Get:15 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [7516 B]
Get:16 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-backports/main Translation-en [4764 B]
Get:17 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [7484 B]
Get:18 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-backports/universe Translation-en [4436 B]
Get:19 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [674 kB] 
Get:20 http://security.ubuntu.com/ubuntu bionic-security/universe Translation-en [224 kB]
Get:21 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [7696 B]
Get:22 http://security.ubuntu.com/ubuntu bionic-security/multiverse Translation-en [2792 B]
Get:23 http://archive.canonical.com/ubuntu bionic/partner amd64 Packages [2288 B]     
Get:24 http://archive.canonical.com/ubuntu bionic/partner Translation-en [1332 B]
Fetched 16.5 MB in 4s (4693 kB/s) 
Reading package lists... Done
efm@controller-0:~$ sudo apt-get install -y nginx
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  grub-pc-bin libnuma1
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip libnginx-mod-http-image-filter
  libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libtiff5 libwebp6 libxpm4 nginx-common nginx-core
Suggested packages:
  libgd-tools fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip libnginx-mod-http-image-filter
  libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libtiff5 libwebp6 libxpm4 nginx nginx-common nginx-core
0 upgraded, 18 newly installed, 0 to remove and 0 not upgraded.
Need to get 2462 kB of archives.
After this operation, 8210 kB of additional disk space will be used.
Get:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libjpeg-turbo8 amd64 1.5.2-0ubuntu5.18.04.4 [110 kB]
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 fonts-dejavu-core all 2.37-1 [1041 kB]
Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 fontconfig-config all 2.12.6-0ubuntu2 [55.8 kB]
Get:4 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 libfontconfig1 amd64 2.12.6-0ubuntu2 [137 kB]
Get:5 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 libjpeg8 amd64 8c-2ubuntu8 [2194 B]
Get:6 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 libjbig0 amd64 2.1-3.1build1 [26.7 kB]
Get:7 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libtiff5 amd64 4.0.9-5ubuntu0.3 [153 kB]
Get:8 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 libwebp6 amd64 0.6.1-2 [185 kB]
Get:9 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 libxpm4 amd64 1:3.5.12-1 [34.0 kB]
Get:10 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgd3 amd64 2.2.5-4ubuntu0.4 [119 kB]
Get:11 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 nginx-common all 1.14.0-0ubuntu1.7 [37.4 kB]
Get:12 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-http-geoip amd64 1.14.0-0ubuntu1.7 [11.2 kB]
Get:13 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-http-image-filter amd64 1.14.0-0ubuntu1.7 [14.6 kB]
Get:14 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-http-xslt-filter amd64 1.14.0-0ubuntu1.7 [13.0 kB]
Get:15 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-mail amd64 1.14.0-0ubuntu1.7 [41.8 kB]
Get:16 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-stream amd64 1.14.0-0ubuntu1.7 [63.7 kB]
Get:17 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 nginx-core amd64 1.14.0-0ubuntu1.7 [413 kB]
Get:18 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates/main amd64 nginx all 1.14.0-0ubuntu1.7 [3596 B]
Fetched 2462 kB in 0s (35.6 MB/s)
Preconfiguring packages ...
Selecting previously unselected package libjpeg-turbo8:amd64.
(Reading database ... 65436 files and directories currently installed.)
Preparing to unpack .../00-libjpeg-turbo8_1.5.2-0ubuntu5.18.04.4_amd64.deb ...
Unpacking libjpeg-turbo8:amd64 (1.5.2-0ubuntu5.18.04.4) ...
Selecting previously unselected package fonts-dejavu-core.
Preparing to unpack .../01-fonts-dejavu-core_2.37-1_all.deb ...
Unpacking fonts-dejavu-core (2.37-1) ...
Selecting previously unselected package fontconfig-config.
Preparing to unpack .../02-fontconfig-config_2.12.6-0ubuntu2_all.deb ...
Unpacking fontconfig-config (2.12.6-0ubuntu2) ...
Selecting previously unselected package libfontconfig1:amd64.
Preparing to unpack .../03-libfontconfig1_2.12.6-0ubuntu2_amd64.deb ...
Unpacking libfontconfig1:amd64 (2.12.6-0ubuntu2) ...
Selecting previously unselected package libjpeg8:amd64.
Preparing to unpack .../04-libjpeg8_8c-2ubuntu8_amd64.deb ...
Unpacking libjpeg8:amd64 (8c-2ubuntu8) ...
Selecting previously unselected package libjbig0:amd64.
Preparing to unpack .../05-libjbig0_2.1-3.1build1_amd64.deb ...
Unpacking libjbig0:amd64 (2.1-3.1build1) ...
Selecting previously unselected package libtiff5:amd64.
Preparing to unpack .../06-libtiff5_4.0.9-5ubuntu0.3_amd64.deb ...
Unpacking libtiff5:amd64 (4.0.9-5ubuntu0.3) ...
Selecting previously unselected package libwebp6:amd64.
Preparing to unpack .../07-libwebp6_0.6.1-2_amd64.deb ...
Unpacking libwebp6:amd64 (0.6.1-2) ...
Selecting previously unselected package libxpm4:amd64.
Preparing to unpack .../08-libxpm4_1%3a3.5.12-1_amd64.deb ...
Unpacking libxpm4:amd64 (1:3.5.12-1) ...
Selecting previously unselected package libgd3:amd64.
Preparing to unpack .../09-libgd3_2.2.5-4ubuntu0.4_amd64.deb ...
Unpacking libgd3:amd64 (2.2.5-4ubuntu0.4) ...
Selecting previously unselected package nginx-common.
Preparing to unpack .../10-nginx-common_1.14.0-0ubuntu1.7_all.deb ...
Unpacking nginx-common (1.14.0-0ubuntu1.7) ...
Selecting previously unselected package libnginx-mod-http-geoip.
Preparing to unpack .../11-libnginx-mod-http-geoip_1.14.0-0ubuntu1.7_amd64.deb ...
Unpacking libnginx-mod-http-geoip (1.14.0-0ubuntu1.7) ...
Selecting previously unselected package libnginx-mod-http-image-filter.
Preparing to unpack .../12-libnginx-mod-http-image-filter_1.14.0-0ubuntu1.7_amd64.deb ...
Unpacking libnginx-mod-http-image-filter (1.14.0-0ubuntu1.7) ...
Selecting previously unselected package libnginx-mod-http-xslt-filter.
Preparing to unpack .../13-libnginx-mod-http-xslt-filter_1.14.0-0ubuntu1.7_amd64.deb ...
Unpacking libnginx-mod-http-xslt-filter (1.14.0-0ubuntu1.7) ...
Selecting previously unselected package libnginx-mod-mail.
Preparing to unpack .../14-libnginx-mod-mail_1.14.0-0ubuntu1.7_amd64.deb ...
Unpacking libnginx-mod-mail (1.14.0-0ubuntu1.7) ...
Selecting previously unselected package libnginx-mod-stream.
Preparing to unpack .../15-libnginx-mod-stream_1.14.0-0ubuntu1.7_amd64.deb ...
Unpacking libnginx-mod-stream (1.14.0-0ubuntu1.7) ...
Selecting previously unselected package nginx-core.
Preparing to unpack .../16-nginx-core_1.14.0-0ubuntu1.7_amd64.deb ...
Unpacking nginx-core (1.14.0-0ubuntu1.7) ...
Selecting previously unselected package nginx.
Preparing to unpack .../17-nginx_1.14.0-0ubuntu1.7_all.deb ...
Unpacking nginx (1.14.0-0ubuntu1.7) ...
Setting up libjbig0:amd64 (2.1-3.1build1) ...
Setting up fonts-dejavu-core (2.37-1) ...
Setting up nginx-common (1.14.0-0ubuntu1.7) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
Setting up libjpeg-turbo8:amd64 (1.5.2-0ubuntu5.18.04.4) ...
Setting up libnginx-mod-mail (1.14.0-0ubuntu1.7) ...
Setting up libxpm4:amd64 (1:3.5.12-1) ...
Setting up libnginx-mod-http-xslt-filter (1.14.0-0ubuntu1.7) ...
Setting up libnginx-mod-http-geoip (1.14.0-0ubuntu1.7) ...
Setting up libwebp6:amd64 (0.6.1-2) ...
Setting up libjpeg8:amd64 (8c-2ubuntu8) ...
Setting up fontconfig-config (2.12.6-0ubuntu2) ...
Setting up libnginx-mod-stream (1.14.0-0ubuntu1.7) ...
Setting up libtiff5:amd64 (4.0.9-5ubuntu0.3) ...
Setting up libfontconfig1:amd64 (2.12.6-0ubuntu2) ...
Setting up libgd3:amd64 (2.2.5-4ubuntu0.4) ...
Setting up libnginx-mod-http-image-filter (1.14.0-0ubuntu1.7) ...
Setting up nginx-core (1.14.0-0ubuntu1.7) ...
Setting up nginx (1.14.0-0ubuntu1.7) ...
Processing triggers for systemd (237-3ubuntu10.41) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ...
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
```

efm@controller-0:~$ {
>   sudo mv kubernetes.default.svc.cluster.local \
>     /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
> 
>   sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
> }

efm@controller-0:~$ sudo systemctl restart nginx
efm@controller-0:~$ sudo systemctl enable nginx
Synchronizing state of nginx.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable nginx

efm@controller-0:~$ kubectl get componentstatuses --kubeconfig admin.kubeconfig
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
etcd-1               Healthy   {"health":"true"}   
controller-manager   Healthy   ok                  
etcd-2               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}   

efm@controller-0:~$ curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Sat, 27 Jun 2020 17:18:45 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 2
Connection: keep-alive
X-Content-Type-Options: nosniff

### RBAC for Kubelet Authorization

```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF

efm@controller-0:~$ cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
> apiVersion: rbac.authorization.k8s.io/v1beta1
> kind: ClusterRole
> metadata:
>   annotations:
>     rbac.authorization.kubernetes.io/autoupdate: "true"
>   labels:
>     kubernetes.io/bootstrapping: rbac-defaults
>   name: system:kube-apiserver-to-kubelet
> rules:
>   - apiGroups:
>       - ""
>     resources:
>       - nodes/proxy
>       - nodes/stats
>       - nodes/log
>       - nodes/spec
>       - nodes/metrics
>     verbs:
>       - "*"
> EOF
clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created

efm@controller-0:~$ cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
> apiVersion: rbac.authorization.k8s.io/v1beta1
> kind: ClusterRoleBinding
> metadata:
>   name: system:kube-apiserver
>   namespace: ""
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: ClusterRole
>   name: system:kube-apiserver-to-kubelet
> subjects:
>   - apiGroup: rbac.authorization.k8s.io
>     kind: User
>     name: kubernetes
> EOF
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created
```

### Kubernetes Frontend Load Balancer

```
efm@controller-0:~$ {
>   KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>     --region $(gcloud config get-value compute/region) \
>     --format 'value(address)')
> 
>   gcloud compute http-health-checks create kubernetes \
>     --description "Kubernetes Health Check" \
>     --host "kubernetes.default.svc.cluster.local" \
>     --request-path "/healthz"
> 
>   gcloud compute firewall-rules create kubernetes-the-hard-way-allow-health-check \
>     --network kubernetes-the-hard-way \
>     --source-ranges 209.85.152.0/22,209.85.204.0/22,35.191.0.0/16 \
>     --allow tcp
> 
>   gcloud compute target-pools create kubernetes-target-pool \
>     --http-health-check kubernetes
> 
>   gcloud compute target-pools add-instances kubernetes-target-pool \
>    --instances controller-0,controller-1,controller-2
> 
>   gcloud compute forwarding-rules create kubernetes-forwarding-rule \
>     --address ${KUBERNETES_PUBLIC_ADDRESS} \
>     --ports 6443 \
>     --region $(gcloud config get-value compute/region) \
>     --target-pool kubernetes-target-pool
> }
(unset)
ERROR: (gcloud.compute.addresses.describe) argument --region: expected one argument
Usage: gcloud compute addresses describe NAME [optional flags]
  optional flags may be  --global | --help | --region

For detailed information on this command and its flags, run:
  gcloud compute addresses describe --help
Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/httpHealthChecks/kubernetes].
NAME        HOST                                  PORT  REQUEST_PATH
kubernetes  kubernetes.default.svc.cluster.local  80    /healthz
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-health-check].     
Creating firewall...done.                                                                                                                                    
NAME                                        NETWORK                  DIRECTION  PRIORITY  ALLOW  DENY  DISABLED
kubernetes-the-hard-way-allow-health-check  kubernetes-the-hard-way  INGRESS    1000      tcp          False
Did you mean region [us-central1] for target pool: 
[kubernetes-target-pool] (Y/n)?  y

Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool].
NAME                    REGION       SESSION_AFFINITY  BACKUP  HEALTH_CHECKS
kubernetes-target-pool  us-central1  NONE                      kubernetes
Did you mean zone [us-central1-b] for instance: [controller-0, 
controller-1, controller-2] (Y/n)?  y

Updated [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool].
(unset)
ERROR: (gcloud.compute.forwarding-rules.create) argument --address: expected one argument
Usage: gcloud compute forwarding-rules create NAME (--backend-service=BACKEND_SERVICE | --target-http-proxy=TARGET_HTTP_PROXY | --target-https-proxy=TARGET_HTTPS_PROXY | --target-instance=TARGET_INSTANCE | --target-pool=TARGET_POOL | --target-ssl-proxy=TARGET_SSL_PROXY | --target-tcp-proxy=TARGET_TCP_PROXY | --target-vpn-gateway=TARGET_VPN_GATEWAY) [optional flags]
  optional flags may be  --address | --address-region | --allow-global-access |
                         --backend-service | --backend-service-region |
                         --description | --global | --global-address |
                         --global-backend-service | --global-target-http-proxy |
                         --global-target-https-proxy | --help | --ip-protocol |
                         --ip-version | --is-mirroring-collector |
                         --load-balancing-scheme | --network | --network-tier |
                         --port-range | --ports | --region | --service-label |
                         --subnet | --subnet-region | --target-http-proxy |
                         --target-http-proxy-region | --target-https-proxy |
                         --target-https-proxy-region | --target-instance |
                         --target-instance-zone | --target-pool |
                         --target-pool-region | --target-ssl-proxy |
                         --target-tcp-proxy | --target-vpn-gateway |
                         --target-vpn-gateway-region

For detailed information on this command and its flags, run:
  gcloud compute forwarding-rules create --help

  efm@controller-1:~$ {
>   KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>     --region $(gcloud config get-value compute/region) \
>     --format 'value(address)')
> 
>   gcloud compute http-health-checks create kubernetes \
>     --description "Kubernetes Health Check" \
>     --host "kubernetes.default.svc.cluster.local" \
>     --request-path "/healthz"
> 
>   gcloud compute firewall-rules create kubernetes-the-hard-way-allow-health-check \
>     --network kubernetes-the-hard-way \
>     --source-ranges 209.85.152.0/22,209.85.204.0/22,35.191.0.0/16 \
>     --allow tcp
> 
>   gcloud compute target-pools create kubernetes-target-pool \
>     --http-health-check kubernetes
> 
>   gcloud compute target-pools add-instances kubernetes-target-pool \
>    --instances controller-0,controller-1,controller-2
> 
>   gcloud compute forwarding-rules create kubernetes-forwarding-rule \
>     --address ${KUBERNETES_PUBLIC_ADDRESS} \
>     --ports 6443 \
>     --region $(gcloud config get-value compute/region) \
>     --target-pool kubernetes-target-pool
> }
(unset)
ERROR: (gcloud.compute.addresses.describe) argument --region: expected one argument
Usage: gcloud compute addresses describe NAME [optional flags]
  optional flags may be  --global | --help | --region

For detailed information on this command and its flags, run:
  gcloud compute addresses describe --help
ERROR: (gcloud.compute.http-health-checks.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/global/httpHealthChecks/kubernetes' already exists

Creating firewall...failed.                                                                                                                                  
ERROR: (gcloud.compute.firewall-rules.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-health-check' already exists

Did you mean region [us-central1] for target pool: 
[kubernetes-target-pool] (Y/n)?  y

ERROR: (gcloud.compute.target-pools.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool' already exists

Did you mean zone [us-central1-b] for instance: [controller-0, 
controller-1, controller-2] (Y/n)?  y

Updated [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool].
(unset)
ERROR: (gcloud.compute.forwarding-rules.create) argument --address: expected one argument
Usage: gcloud compute forwarding-rules create NAME (--backend-service=BACKEND_SERVICE | --target-http-proxy=TARGET_HTTP_PROXY | --target-https-proxy=TARGET_HTTPS_PROXY | --target-instance=TARGET_INSTANCE | --target-pool=TARGET_POOL | --target-ssl-proxy=TARGET_SSL_PROXY | --target-tcp-proxy=TARGET_TCP_PROXY | --target-vpn-gateway=TARGET_VPN_GATEWAY) [optional flags]
  optional flags may be  --address | --address-region | --allow-global-access |
                         --backend-service | --backend-service-region |
                         --description | --global | --global-address |
                         --global-backend-service | --global-target-http-proxy |
                         --global-target-https-proxy | --help | --ip-protocol |
                         --ip-version | --is-mirroring-collector |
                         --load-balancing-scheme | --network | --network-tier |
                         --port-range | --ports | --region | --service-label |
                         --subnet | --subnet-region | --target-http-proxy |
                         --target-http-proxy-region | --target-https-proxy |
                         --target-https-proxy-region | --target-instance |
                         --target-instance-zone | --target-pool |
                         --target-pool-region | --target-ssl-proxy |
                         --target-tcp-proxy | --target-vpn-gateway |
                         --target-vpn-gateway-region

For detailed information on this command and its flags, run:
  gcloud compute forwarding-rules create --help

efm@controller-2:~$ {
>   KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>     --region $(gcloud config get-value compute/region) \
>     --format 'value(address)')
> 
>   gcloud compute http-health-checks create kubernetes \
>     --description "Kubernetes Health Check" \
>     --host "kubernetes.default.svc.cluster.local" \
>     --request-path "/healthz"
> 
>   gcloud compute firewall-rules create kubernetes-the-hard-way-allow-health-check \
>     --network kubernetes-the-hard-way \
>     --source-ranges 209.85.152.0/22,209.85.204.0/22,35.191.0.0/16 \
>     --allow tcp
> 
>   gcloud compute target-pools create kubernetes-target-pool \
>     --http-health-check kubernetes
> 
>   gcloud compute target-pools add-instances kubernetes-target-pool \
>    --instances controller-0,controller-1,controller-2
> 
>   gcloud compute forwarding-rules create kubernetes-forwarding-rule \
>     --address ${KUBERNETES_PUBLIC_ADDRESS} \
>     --ports 6443 \
>     --region $(gcloud config get-value compute/region) \
>     --target-pool kubernetes-target-pool
> }
(unset)
ERROR: (gcloud.compute.addresses.describe) argument --region: expected one argument
Usage: gcloud compute addresses describe NAME [optional flags]
  optional flags may be  --global | --help | --region

For detailed information on this command and its flags, run:
  gcloud compute addresses describe --help
ERROR: (gcloud.compute.http-health-checks.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/global/httpHealthChecks/kubernetes' already exists

Creating firewall...failed.                                                                                                                                  
ERROR: (gcloud.compute.firewall-rules.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-health-check' already exists

Did you mean region [us-central1] for target pool: 
[kubernetes-target-pool] (Y/n)?  y

ERROR: (gcloud.compute.target-pools.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool' already exists

Did you mean zone [us-central1-b] for instance: [controller-0, 
controller-1, controller-2] (Y/n)?  y

Updated [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool].
(unset)
ERROR: (gcloud.compute.forwarding-rules.create) argument --address: expected one argument
Usage: gcloud compute forwarding-rules create NAME (--backend-service=BACKEND_SERVICE | --target-http-proxy=TARGET_HTTP_PROXY | --target-https-proxy=TARGET_HTTPS_PROXY | --target-instance=TARGET_INSTANCE | --target-pool=TARGET_POOL | --target-ssl-proxy=TARGET_SSL_PROXY | --target-tcp-proxy=TARGET_TCP_PROXY | --target-vpn-gateway=TARGET_VPN_GATEWAY) [optional flags]
  optional flags may be  --address | --address-region | --allow-global-access |
                         --backend-service | --backend-service-region |
                         --description | --global | --global-address |
                         --global-backend-service | --global-target-http-proxy |
                         --global-target-https-proxy | --help | --ip-protocol |
                         --ip-version | --is-mirroring-collector |
                         --load-balancing-scheme | --network | --network-tier |
                         --port-range | --ports | --region | --service-label |
                         --subnet | --subnet-region | --target-http-proxy |
                         --target-http-proxy-region | --target-https-proxy |
                         --target-https-proxy-region | --target-instance |
                         --target-instance-zone | --target-pool |
                         --target-pool-region | --target-ssl-proxy |
                         --target-tcp-proxy | --target-vpn-gateway |
                         --target-vpn-gateway-region

For detailed information on this command and its flags, run:
  gcloud compute forwarding-rules create --help

```

### Verify

Run this on my laptop

```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

```

To fix  the region errors, in each controller I did that, which was the wrong thing

```
 gcloud  config set compute/region  'us-central1'
```

```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>   --region $(gcloud config get-value compute/region) \
>   --format 'value(address)')
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ curl --cacert ca.pem https://${KUBERNETES_PUBLIC_ADDRESS}:6443/version
```
That command timed out, and I don't know why. My guess is that my gcloud init connection timed out after 3 hours.

Took lunch, came benchmark.md

```
gcloud init
```

To login, set the project and the compute region.

Re-opened a connection to each of  the controllers-{0,1,2}
Checked health on all 3 controllers

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
scheduler            Healthy   ok                  
etcd-2               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   
```

Then checked the nginx health proxy.

```
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz

```

### RBAC

Run on only one of the controllers.

```
efm@controller-0:~$ cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
> apiVersion: rbac.authorization.k8s.io/v1beta1
> kind: ClusterRole
> metadata:
>   annotations:
>     rbac.authorization.kubernetes.io/autoupdate: "true"
>   labels:
>     kubernetes.io/bootstrapping: rbac-defaults
>   name: system:kube-apiserver-to-kubelet
> rules:
>   - apiGroups:
>       - ""
>     resources:
>       - nodes/proxy
>       - nodes/stats
>       - nodes/log
>       - nodes/spec
>       - nodes/metrics
>     verbs:
>       - "*"
> EOF
clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet unchanged
```

I had run that before lunch, so unchanged is correct.

```
efm@controller-0:~$ cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
> apiVersion: rbac.authorization.k8s.io/v1beta1
> kind: ClusterRoleBinding
> metadata:
>   name: system:kube-apiserver
>   namespace: ""
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: ClusterRole
>   name: system:kube-apiserver-to-kubelet
> subjects:
>   - apiGroup: rbac.authorization.k8s.io
>     kind: User
>     name: kubernetes
> EOF
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver unchanged
```

That also looks ok.

Back to running commands on my laptop, which was used to provision the instances.

```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl get componentstatuses --kubeconfig admin.kubeconfig
The connection to the server 127.0.0.1:6443 was refused - did you specify the right host or port?
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ {
>   KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>     --region $(gcloud config get-value compute/region) \
>     --format 'value(address)')
> 
>   gcloud compute http-health-checks create kubernetes \
>     --description "Kubernetes Health Check" \
>     --host "kubernetes.default.svc.cluster.local" \
>     --request-path "/healthz"
> 
>   gcloud compute firewall-rules create kubernetes-the-hard-way-allow-health-check \
>     --network kubernetes-the-hard-way \
>     --source-ranges 209.85.152.0/22,209.85.204.0/22,35.191.0.0/16 \
>     --allow tcp
> 
>   gcloud compute target-pools create kubernetes-target-pool \
>     --http-health-check kubernetes
> 
>   gcloud compute target-pools add-instances kubernetes-target-pool \
>    --instances controller-0,controller-1,controller-2
> 
>   gcloud compute forwarding-rules create kubernetes-forwarding-rule \
>     --address ${KUBERNETES_PUBLIC_ADDRESS} \
>     --ports 6443 \
>     --region $(gcloud config get-value compute/region) \
>     --target-pool kubernetes-target-pool
> }
ERROR: (gcloud.compute.http-health-checks.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/global/httpHealthChecks/kubernetes' already exists

Creating firewall...failed.                                                                                                                                  
ERROR: (gcloud.compute.firewall-rules.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-health-check' already exists

ERROR: (gcloud.compute.target-pools.create) Could not fetch resource:
 - The resource 'projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool' already exists

Updated [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool].
Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/forwardingRules/kubernetes-forwarding-rule].

#### Verify

```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

  efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>   --region $(gcloud config get-value compute/region) \
>   --format 'value(address)')

curl --cacert ca.pem https://${KUBERNETES_PUBLIC_ADDRESS}:6443/version
{
  "major": "1",
  "minor": "15",
  "gitVersion": "v1.15.3",
  "gitCommit": "2d3c76f9091b6bec110a5e63777c332469e0cba2",
  "gitTreeState": "clean",
  "buildDate": "2019-08-19T11:05:50Z",
  "goVersion": "go1.12.9",
  "compiler": "gc",
  "platform": "linux/amd64"
```

##  Bootstrapping the worker nodes

In this lab you will bootstrap three Kubernetes worker nodes. The following components will be installed on each node: runc, container networking plugins, containerd, kubelet, and kube-proxy.

Run on all three controllers-{0,1,2} separately

### Provision a worker node

```
efm@controller-0:~$ {
>   sudo apt-get update
>   sudo apt-get -y install socat conntrack ipset
> }
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]  
Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:4 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                                          
Hit:5 http://archive.canonical.com/ubuntu bionic InRelease                                                           
Fetched 252 kB in 0s (545 kB/s)                                                     
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  grub-pc-bin libnuma1
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libipset3
The following NEW packages will be installed:
  conntrack ipset libipset3 socat
0 upgraded, 4 newly installed, 0 to remove and 0 not upgraded.
Need to get 450 kB of archives.
After this operation, 1560 kB of additional disk space will be used.
Get:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 conntrack amd64 1:1.4.4+snapshot20161117-6ubuntu2 [30.6 kB]
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 libipset3 amd64 6.34-1 [43.9 kB]
Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 ipset amd64 6.34-1 [33.7 kB]
Get:4 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic/main amd64 socat amd64 1.7.3.2-2ubuntu2 [342 kB]
Fetched 450 kB in 0s (19.4 MB/s)
Selecting previously unselected package conntrack.
(Reading database ... 65672 files and directories currently installed.)
Preparing to unpack .../conntrack_1%3a1.4.4+snapshot20161117-6ubuntu2_amd64.deb ...
Unpacking conntrack (1:1.4.4+snapshot20161117-6ubuntu2) ...
Selecting previously unselected package libipset3:amd64.
Preparing to unpack .../libipset3_6.34-1_amd64.deb ...
Unpacking libipset3:amd64 (6.34-1) ...
Selecting previously unselected package ipset.
Preparing to unpack .../ipset_6.34-1_amd64.deb ...
Unpacking ipset (6.34-1) ...
Selecting previously unselected package socat.
Preparing to unpack .../socat_1.7.3.2-2ubuntu2_amd64.deb ...
Unpacking socat (1.7.3.2-2ubuntu2) ...
Setting up conntrack (1:1.4.4+snapshot20161117-6ubuntu2) ...
Setting up socat (1.7.3.2-2ubuntu2) ...
Setting up libipset3:amd64 (6.34-1) ...
Setting up ipset (6.34-1) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

### Disable Swap

```
sudo swapon --show
```

Install the software

```
efm@controller-0:~$ wget -q --show-progress --https-only --timestamping \
>   https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.15.0/crictl-v1.15.0-linux-amd64.tar.gz \
>   https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.amd64 \
>   https://github.com/containernetworking/plugins/releases/download/v0.8.2/cni-plugins-linux-amd64-v0.8.2.tgz \
>   https://github.com/containerd/containerd/releases/download/v1.2.9/containerd-1.2.9.linux-amd64.tar.gz \
>   https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl \
>   https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy \
>   https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet
crictl-v1.15.0-linux-amd64.tar.gz       100%[=============================================================================>]  11.50M  38.4MB/s    in 0.3s    
runc.amd64                              100%[=============================================================================>]   9.79M  33.9MB/s    in 0.3s    
cni-plugins-linux-amd64-v0.8.2.tgz      100%[=============================================================================>]  34.96M  65.4MB/s    in 0.5s    
containerd-1.2.9.linux-amd64.tar.gz     100%[=============================================================================>]  32.96M  62.5MB/s    in 0.5s    
kubectl                                 100%[=============================================================================>]  40.99M   132MB/s    in 0.3s    
kube-proxy                              100%[=============================================================================>]  35.27M   103MB/s    in 0.3s    
kubelet                                 100%[=============================================================================>] 114.10M   112MB/s    in 1.0s    
```

```
efm@controller-0:~$ sudo mkdir -p \
>   /etc/cni/net.d \
>   /opt/cni/bin \
>   /var/lib/kubelet \
>   /var/lib/kube-proxy \
>   /var/lib/kubernetes \
>   /var/run/kubernetes
```

```
efm@controller-0:~$ {
>   mkdir containerd
>   tar -xvf crictl-v1.15.0-linux-amd64.tar.gz
>   tar -xvf containerd-1.2.9.linux-amd64.tar.gz -C containerd
>   sudo tar -xvf cni-plugins-linux-amd64-v0.8.2.tgz -C /opt/cni/bin/
>   sudo mv runc.amd64 runc
>   chmod +x crictl kubectl kube-proxy kubelet runc 
>   sudo mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
>   sudo mv containerd/bin/* /bin/
> }
crictl
bin/
bin/containerd-shim-runc-v1
bin/ctr
bin/containerd
bin/containerd-stress
bin/containerd-shim
./
./flannel
./ptp
./host-local
./firewall
./portmap
./tuning
./vlan
./host-device
./bandwidth
./sbr
./static
./dhcp
./ipvlan
./macvlan
./loopback
./bridge
```

### Configure CNI networking

```
POD_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/pod-cidr)
```

```
efm@controller-0:~$ cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
> {
>     "cniVersion": "0.3.1",
>     "name": "bridge",
>     "type": "bridge",
>     "bridge": "cnio0",
>     "isGateway": true,
>     "ipMasq": true,
>     "ipam": {
>         "type": "host-local",
>         "ranges": [
>           [{"subnet": "${POD_CIDR}"}]
>         ],
>         "routes": [{"dst": "0.0.0.0/0"}]
>     }
> }
> EOF
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "<!DOCTYPE html>
<html lang=en>
  <meta charset=utf-8>
  <meta name=viewport content="initial-scale=1, minimum-scale=1, width=device-width">
  <title>Error 404 (Not Found)!!1</title>
  <style>
    *{margin:0;padding:0}html,code{font:15px/22px arial,sans-serif}html{background:#fff;color:#222;padding:15px}body{margin:7% auto 0;max-width:390px;min-height:180px;padding:30px 0 15px}* > body{background:url(//www.google.com/images/errors/robot.png) 100% 5px no-repeat;padding-right:205px}p{margin:11px 0 22px;overflow:hidden}ins{color:#777;text-decoration:none}a img{border:0}@media screen and (max-width:772px){body{background:none;margin-top:0;max-width:none;padding-right:0}}#logo{background:url(//www.google.com/images/branding/googlelogo/1x/googlelogo_color_150x54dp.png) no-repeat;margin-left:-5px}@media only screen and (min-resolution:192dpi){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat 0% 0%/100% 100%;-moz-border-image:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) 0}}@media only screen and (-webkit-min-device-pixel-ratio:2){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat;-webkit-background-size:100% 100%}}#logo{display:inline-block;height:54px;width:150px}
  </style>
  <a href=//www.google.com/><span id=logo aria-label=Google></span></a>
  <p><b>404.</b> <ins>That’s an error.</ins>
  <p>The requested URL <code>/computeMetadata/v1/instance/attributes/pod-cidr</code> was not found on this server.  <ins>That’s all we know.</ins>"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
```

My error was not switching to the worker nodes.

```
efm@worker-0:~$ {
>   sudo apt-get update
>   sudo apt-get -y install socat conntrack ipset
> }
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]                   
...
```

efm@worker-0:~$ wget -q --show-progress --https-only --timestamping \
>   https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.15.0/crictl-v1.15.0-linux-amd64.tar.gz \
>   https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.amd64 \
>   https://github.com/containernetworking/plugins/releases/download/v0.8.2/cni-plugins-linux-amd64-v0.8.2.tgz \
>   https://github.com/containerd/containerd/releases/download/v1.2.9/containerd-1.2.9.linux-amd64.tar.gz \
>   https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl \
>   https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy \
>   https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet
crictl-v1.15.0-linux-amd64.tar.gz       100%[=============================================================================>]  11.50M  37.7MB/s    in 0.3s    
runc.amd64                              100%[=============================================================================>]   9.79M  35.2MB/s    in 0.3s    
cni-plugins-linux-amd64-v0.8.2.tgz      100%[=============================================================================>]  34.96M  65.2MB/s    in 0.5s    
containerd-1.2.9.linux-amd64.tar.gz     100%[=============================================================================>]  32.96M  64.7MB/s    in 0.5s    
kubectl                                 100%[=============================================================================>]  40.99M   113MB/s    in 0.4s    
kube-proxy                              100%[=============================================================================>]  35.27M   185MB/s    in 0.2s    
kubelet                                 100%[=============================================================================>] 114.10M   207MB/s    in 0.6s    
```

```
efm@worker-0:~$ sudo mkdir -p \
>   /etc/cni/net.d \
>   /opt/cni/bin \
>   /var/lib/kubelet \
>   /var/lib/kube-proxy \
>   /var/lib/kubernetes \
>   /var/run/kubernetes
```

```
efm@worker-0:~$ {
>   mkdir containerd
>   tar -xvf crictl-v1.15.0-linux-amd64.tar.gz
>   tar -xvf containerd-1.2.9.linux-amd64.tar.gz -C containerd
>   sudo tar -xvf cni-plugins-linux-amd64-v0.8.2.tgz -C /opt/cni/bin/
>   sudo mv runc.amd64 runc
>   chmod +x crictl kubectl kube-proxy kubelet runc 
>   sudo mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
>   sudo mv containerd/bin/* /bin/
> }
crictl
bin/
bin/containerd-shim-runc-v1
bin/ctr
bin/containerd
bin/containerd-stress
bin/containerd-shim
./
./flannel
./ptp
./host-local
./firewall
./portmap
./tuning
./vlan
./host-device
./bandwidth
./sbr
./static
./dhcp
./ipvlan
./macvlan
./loopback
./bridge
```

```
efm@worker-0:~$ POD_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
>   http://metadata.google.internal/computeMetadata/v1/instance/attributes/pod-cidr)
```

```
efm@worker-0:~$ cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
> {
>     "cniVersion": "0.3.1",
>     "name": "bridge",
>     "type": "bridge",
>     "bridge": "cnio0",
>     "isGateway": true,
>     "ipMasq": true,
>     "ipam": {
>         "type": "host-local",
>         "ranges": [
>           [{"subnet": "${POD_CIDR}"}]
>         ],
>         "routes": [{"dst": "0.0.0.0/0"}]
>     }
> }
> EOF
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "10.200.0.0/24"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
```

```
efm@worker-0:~$ cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
> {
>     "cniVersion": "0.3.1",
>     "name": "lo",
>     "type": "loopback"
> }
> EOF
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
```

#### Configure containerd

```
efm@worker-0:~$ cat << EOF | sudo tee /etc/containerd/config.toml
> [plugins]
>   [plugins.cri.containerd]
>     snapshotter = "overlayfs"
>     [plugins.cri.containerd.default_runtime]
>       runtime_type = "io.containerd.runtime.v1.linux"
>       runtime_engine = "/usr/local/bin/runc"
>       runtime_root = ""
> EOF
tee: /etc/containerd/config.toml: No such file or directory
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
```

```
efm@worker-0:~$ cat <<EOF | sudo tee /etc/systemd/system/containerd.service
> [Unit]
> Description=containerd container runtime
> Documentation=https://containerd.io
> After=network.target
> 
> [Service]
> ExecStartPre=/sbin/modprobe overlay
> ExecStart=/bin/containerd
> Restart=always
> RestartSec=5
> Delegate=yes
> KillMode=process
> OOMScoreAdjust=-999
> LimitNOFILE=1048576
> LimitNPROC=infinity
> LimitCORE=infinity
> 
> [Install]
> WantedBy=multi-user.target
> EOF
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
```

### Configure the kubelet

```
efm@worker-0:~$ {
>   sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
>   sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
>   sudo mv ca.pem /var/lib/kubernetes/
> }
```

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

```
efm@worker-0:~$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
> kind: KubeletConfiguration
> apiVersion: kubelet.config.k8s.io/v1beta1
> authentication:
>   anonymous:
>     enabled: false
>   webhook:
>     enabled: true
>   x509:
>     clientCAFile: "/var/lib/kubernetes/ca.pem"
> authorization:
>   mode: Webhook
> clusterDomain: "cluster.local"
> clusterDNS:
>   - "10.32.0.10"
> podCIDR: "${POD_CIDR}"
> resolvConf: "/run/systemd/resolve/resolv.conf"
> runtimeRequestTimeout: "15m"
> tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
> tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
> EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "10.200.0.0/24"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/worker-0.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/worker-0-key.pem
```

```
efm@worker-0:~$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
> [Unit]
> Description=Kubernetes Kubelet
> Documentation=https://github.com/kubernetes/kubernetes
> After=containerd.service
> Requires=containerd.service
> 
> [Service]
> ExecStart=/usr/local/bin/kubelet \\
>   --config=/var/lib/kubelet/kubelet-config.yaml \\
>   --container-runtime=remote \\
>   --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
>   --image-pull-progress-deadline=2m \\
>   --kubeconfig=/var/lib/kubelet/kubeconfig \\
>   --network-plugin=cni \\
>   --register-node=true \\
>   --v=2
> Restart=on-failure
> RestartSec=5
> 
> [Install]
> WantedBy=multi-user.target
> EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/kubelet-config.yaml \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```
efm@worker-0:~$ sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

```
efm@worker-0:~$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
> kind: KubeProxyConfiguration
> apiVersion: kubeproxy.config.k8s.io/v1alpha1
> clientConnection:
>   kubeconfig: "/var/lib/kube-proxy/kubeconfig"
> mode: "iptables"
> clusterCIDR: "10.200.0.0/16"
> EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
```

```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```

```
efm@worker-0:~$ {
>   sudo systemctl daemon-reload
>   sudo systemctl enable containerd kubelet kube-proxy
>   sudo systemctl start containerd kubelet kube-proxy
> }
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /etc/systemd/system/containerd.service.
Created symlink /etc/systemd/system/multi-user.target.wants/kubelet.service → /etc/systemd/system/kubelet.service.
Created symlink /etc/systemd/system/multi-user.target.wants/kube-proxy.service → /etc/systemd/system/kube-proxy.service.
```

## Verify worker nodes

Back to laptop to verify

```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ gcloud compute ssh controller-0 \
>   --command "kubectl get nodes --kubeconfig admin.kubeconfig"
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
NAME       STATUS   ROLES    AGE     VERSION
worker-0   Ready    <none>   6m23s   v1.15.3
worker-1   Ready    <none>   45s     v1.15.3
worker-2   Ready    <none>   41s     v1.15.3
```


## Configuring kubectl for remote access

From my laptop, which I used to create the admin certs

```
{
  KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
    --region $(gcloud config get-value compute/region) \
    --format 'value(address)')

  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```


```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ {
>   KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>     --region $(gcloud config get-value compute/region) \
>     --format 'value(address)')
> 
>   kubectl config set-cluster kubernetes-the-hard-way \
>     --certificate-authority=ca.pem \
>     --embed-certs=true \
>     --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443
> 
>   kubectl config set-credentials admin \
>     --client-certificate=admin.pem \
>     --client-key=admin-key.pem
> 
>   kubectl config set-context kubernetes-the-hard-way \
>     --cluster=kubernetes-the-hard-way \
>     --user=admin
> 
>   kubectl config use-context kubernetes-the-hard-way
> }
Cluster "kubernetes-the-hard-way" set.
User "admin" set.
Context "kubernetes-the-hard-way" created.
Switched to context "kubernetes-the-hard-way".
```

```
kubectl get componentstatuses
NAME                 AGE
controller-manager   <unknown>
scheduler            <unknown>
etcd-0               <unknown>
etcd-2               <unknown>
etcd-1               <unknown>
```

```
kubectl get nodes
```

## Provisioning Pod Network Routes

On my laptop

```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute instances describe ${instance} \
    --format 'value[separator=" "](networkInterfaces[0].networkIP,metadata.items[0].value)'
done
```

```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ for i in 0 1 2; do
>   gcloud compute routes create kubernetes-route-10-200-${i}-0-24 \
>     --network kubernetes-the-hard-way \
>     --next-hop-address 10.240.0.2${i} \
>     --destination-range 10.200.${i}.0/24
> done
Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/routes/kubernetes-route-10-200-0-0-24].
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP     PRIORITY
kubernetes-route-10-200-0-0-24  kubernetes-the-hard-way  10.200.0.0/24  10.240.0.20  1000
Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/routes/kubernetes-route-10-200-1-0-24].
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP     PRIORITY
kubernetes-route-10-200-1-0-24  kubernetes-the-hard-way  10.200.1.0/24  10.240.0.21  1000
Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/routes/kubernetes-route-10-200-2-0-24].
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP     PRIORITY
kubernetes-route-10-200-2-0-24  kubernetes-the-hard-way  10.200.2.0/24  10.240.0.22  1000
```

```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ gcloud compute routes list --filter "network: kubernetes-the-hard-way"
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP                  PRIORITY
default-route-0729d983e716a456  kubernetes-the-hard-way  0.0.0.0/0      default-internet-gateway  1000
default-route-ff5e9b7a81dfd708  kubernetes-the-hard-way  10.240.0.0/24  kubernetes-the-hard-way   0
kubernetes-route-10-200-0-0-24  kubernetes-the-hard-way  10.200.0.0/24  10.240.0.20               1000
kubernetes-route-10-200-1-0-24  kubernetes-the-hard-way  10.200.1.0/24  10.240.0.21               1000
kubernetes-route-10-200-2-0-24  kubernetes-the-hard-way  10.200.2.0/24  10.240.0.22               1000
```

## Deploying the DNS cluster add<!-- markdownlint-configure-file 

```

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl get pods -l k8s-app=kube-dns -n kube-system
NAME                     READY   STATUS    RESTARTS   AGE
coredns-5fb99965-bb8j7   1/1     Running   0          23s
coredns-5fb99965-c7794   1/1     Running   0          23s

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl run --generator=run-pod/v1 busybox --image=busybox:1.28 --command -- sleep 3600
Flag --generator has been deprecated, has no effect and will be removed in the future.
pod/busybox created

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl get pods -l run=busybox
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          26s

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl exec -ti $POD_NAME -- nslookup kubernetes
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local

```

# Verify the whole k8s

```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl create secret generic kubernetes-the-hard-way \
>   --from-literal="mykey=mydata"
secret/kubernetes-the-hard-way created

gcloud compute ssh controller-0 \
  --command "sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"

  efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ gcloud compute ssh controller-0 \
>   --command "sudo ETCDCTL_API=3 etcdctl get \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem\
>   /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
Enter passphrase for key '/home/efm/.ssh/google_compute_engine': 
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a 8a 8a ea d4 1e ad 13  |:v1:key1:.......|
00000050  d4 78 47 53 9d 12 56 56  5f 1c 1f 34 4d 58 a9 b6  |.xGS..VV_..4MX..|
00000060  ea d3 18 bf 6c ca 67 89  64 10 09 ea be db 52 0d  |....l.g.d.....R.|
00000080  b6 76 6f e9 64 a2 f2 1f  ae 84 2e 2b 41 7e b6 99  |.vo.d......+A~..|
00000090  f9 6a 5d 12 d9 2f 52 4d  3a f3 b0 5c 6f 80 4e 5a  |.j]../RM:..\o.NZ|
000000a0  d3 57 d4 43 be f5 32 9e  d4 d7 82 49 2e 95 96 f4  |.W.C..2....I....|
000000b0  a8 21 81 c0 5c 87 e7 73  01 d3 25 58 b6 ac 85 68  |.!..\..s..%X...h|
000000c0  de b9 8c b3 82 be 59 07  1d fe 1b ea 0f ca b8 dc  |......Y.........|
000000d0  44 bd d1 f5 f7 7d 7a 51  7e c7 dc 4f 2a 35 44 56  |D....}zQ~..O*5DV|
000000e0  9a 42 e0 df 43 19 32 dd  1a 0a                    |.B..C.2...|
000000ea

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl get pods -l app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-554b9c67f9-p9s2s   1/1     Running   0          30s


efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl port-forward $POD_NAME 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
--
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ curl --head http://127.0.0.1:8080
HTTP/1.1 200 OK
Server: nginx/1.19.0
Date: Sat, 27 Jun 2020 20:16:23 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 26 May 2020 15:00:20 GMT
Connection: keep-alive
ETag: "5ecd2f04-264"
Accept-Ranges: bytes
--
```

```
efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl logs $POD_NAME
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
127.0.0.1 - - [27/Jun/2020:20:16:23 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.58.0" "-"

kubectl exec -ti $POD_NAME -- nginx -v

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ kubectl exec -ti $POD_NAME -- nginx -v
nginx version: nginx/1.19.0

kubectl expose deployment nginx --port 80 --type NodePort

NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ gcloud compute firewall-rules create kubernetes-the-hard-way-allow-nginx-service \
>   --allow=tcp:${NODE_PORT} \
>   --network kubernetes-the-hard-way
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-nginx-service].    
Creating firewall...done.                                                                                                                                    
NAME                                         NETWORK                  DIRECTION  PRIORITY  ALLOW      DENY  DISABLED
kubernetes-the-hard-way-allow-nginx-service  kubernetes-the-hard-way  INGRESS    1000      tcp:30943        False

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ EXTERNAL_IP=$(gcloud compute instances describe worker-0 \
>   --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

curl -I http://${EXTERNAL_IP}:${NODE_PORT}
```


## Tidy up

```
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ gcloud -q compute instances delete \
>   controller-0 controller-1 controller-2 \
>   worker-0 worker-1 worker-2 \
>   --zone $(gcloud config get-value compute/zone)
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/instances/controller-0].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/instances/controller-1].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/instances/controller-2].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/instances/worker-0].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/instances/worker-1].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/zones/us-central1-b/instances/worker-2].

{
  gcloud -q compute forwarding-rules delete kubernetes-forwarding-rule \
    --region $(gcloud config get-value compute/region)

  gcloud -q compute target-pools delete kubernetes-target-pool

  gcloud -q compute http-health-checks delete kubernetes

  gcloud -q compute addresses delete kubernetes-the-hard-way
}
efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ {
>   gcloud -q compute forwarding-rules delete kubernetes-forwarding-rule \
>     --region $(gcloud config get-value compute/region)
> 
>   gcloud -q compute target-pools delete kubernetes-target-pool
> 
>   gcloud -q compute http-health-checks delete kubernetes
> 
>   gcloud -q compute addresses delete kubernetes-the-hard-way
> }
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/forwardingRules/kubernetes-forwarding-rule].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/targetPools/kubernetes-target-pool].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/httpHealthChecks/kubernetes].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/addresses/kubernetes-the-hard-way].

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ gcloud -q compute firewall-rules delete \
>   kubernetes-the-hard-way-allow-nginx-service \
>   kubernetes-the-hard-way-allow-internal \
>   kubernetes-the-hard-way-allow-external \
>   kubernetes-the-hard-way-allow-health-check
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-nginx-service].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-internal].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-external].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/firewalls/kubernetes-the-hard-way-allow-health-check].

efm@efm:~/Development/SysADmin/Kubernetesk8s/kubernetes-the-hard-way$ {
>   gcloud -q compute routes delete \
>     kubernetes-route-10-200-0-0-24 \
>     kubernetes-route-10-200-1-0-24 \
>     kubernetes-route-10-200-2-0-24
> 
>   gcloud -q compute networks subnets delete kubernetes
> 
>   gcloud -q compute networks delete kubernetes-the-hard-way
> }
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/routes/kubernetes-route-10-200-0-0-24].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/routes/kubernetes-route-10-200-1-0-24].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/routes/kubernetes-route-10-200-2-0-24].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/regions/us-central1/subnetworks/kubernetes].
Deleted [https://www.googleapis.com/compute/v1/projects/k8sthw-280616/global/networks/kubernetes-the-hard-way].
```




























































