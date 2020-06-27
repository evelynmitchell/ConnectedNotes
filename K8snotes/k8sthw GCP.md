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

```
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

































