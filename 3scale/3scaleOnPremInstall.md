# 3scale On Premise Installation Steps

## 3scale Deployment Options: 
3scale deployments can be of 2 types broadly speaking:

__Option 1:__ Gateway (installed on baremetal, Docker or Openshift) that talks to the SaaS API manager. Every participant should have an account on the [3scale.net](http://www.3scale.net) domain

__Option 2:__ Fully On-Prem. In this, all components of 3scale are deployed on Openshift: Gateway and API Manager (which includes the Developer Portal)

__Note__: The instructions below are for Option 2. above, namely: deploy 3scale fully on-prem (AMP)
The official install guide is here: https://support.3scale.net/guides/infrastructure/onpremises20-installation

The below is a cheat sheet.

## Step 1: Check that your cluster is up and running and that you can login to the Openshift console

## Step 2: Verify that you have free PVs that dont have PV claims on them.
If you DONT HAVE free PVs, do the following steps..

On [day 1](https://github.com/RedHatWorkshops/openshiftv3-ops-workshop/blob/master/standing_up_hosts_on_ec2.md) you have setup an openshift cluster that installed an NFS Server on the master.  This NFS server should have been mapped to /exports. 
Log into your server and check your /exports folder

```
ls /exports

```
 
 output should be similar to the below:

```
etcd  logging  logging-es-ops  metrics  registry  vol1  vol2  vol3  vol4

```

Let us create a few volumes on this folder.

```
sudo bash
mkdir -p /exports/vol5
mkdir -p /exports/vol6
mkdir -p /exports/vol7
mkdir -p /exports/vol8

```
Change the ownership to nfsnobody and give 777 permissions

```
chown nfsnobody:nfsnobody /exports/vol5 /exports/vol6 /exports/vol7 /exports/vol8 
chmod 777 /exports/vol5 /exports/vol6 /exports/vol7 /exports/vol8 

```
You would want to declare these folders that you just created as NFS exports
The current exports are created by ansible playbook that installed openshift and are listed in this file

```
ls /etc/exports.d openshift-ansible.exports
```
While we can edit this file to add your exports, let us create a separate file instead and edit the following entries

```
vi /etc/exports.d/veer.exports

/exports/vol5 *(rw,root_squash)
/exports/vol6 *(rw,root_squash)
/exports/vol7 *(rw,root_squash)
/exports/vol8 *(rw,root_squash)

```
save the file and exit vi

run exportfs to update your NFS and restart NFS

```
exportfs -ra
systemctl restart nfs

```
verify that the exports are available 

```
cat /var/lib/nfs/etab

/exports/vol8	*(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,root_squash,no_all_squash)
/exports/vol7	*(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,root_squash,no_all_squash)
/exports/vol5	*(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,root_squash,no_all_squash)
/exports/vol6	*(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,root_squash,no_all_squash)


```
should show you the entries for your exports.

```
vi pv.yml

apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
  name: pv001 
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: /exports/vol1     
    server: 10.0.0.86 <CHANGE THIS TO YOUR MASTER'S INTERNAL IP>
  persistentVolumeReclaimPolicy: Recycle

```

ip address above is the private ip of the master as NFS is running on the master. We create 4 pvs with this template: 3 with ReadWriteOnce and 1 with ReadWriteMany.
  
You can edit the file each time or run it from commandline with sed as follows

```
sed -e s/vol1/vol5/ -e s/pv001/pv005/ pv.yml | oc create -f -
sed -e s/vol1/vol6/ -e s/pv001/pv006/ -e s/ReadWriteMany/ReadWriteOnce/ pv.yml | oc create -f -
sed -e s/vol1/vol7/ -e s/pv001/pv007/ -e s/ReadWriteMany/ReadWriteOnce/ pv.yml | oc create -f -
sed -e s/vol1/vol8/ -e s/pv001/pv008/ -e s/ReadWriteMany/ReadWriteOnce/ pv.yml | oc create -f -

```
Run `oc get pv` to verify your PVs are available

```
pv005             1Gi        RWX           Recycle         Available
pv006             1Gi        RWO           Recycle         Available                                                           pv007             1Gi        RWO           Recycle         Available                                                           pv008             1Gi        RWO           Recycle         Available                                                           
```

## Step 3: Create a new project either via the openshift console or using the oc cli

```oc login``` (developer account will suffice, dont need admin)

```oc new-project <project_name>``` say 3scaleamp

## Step 4: Create a new app with the AMP template

Note that in the install guide, you access the template from a 3scale rpm but for this lab, we will make it available here.
The AMP template installs:
1. Two built-in API gateways
2. One AMP admin portal and developer portal with persistent storage

copy the [amp.yml](https://raw.githubusercontent.com/kasriniv/openshiftv3-ops-workshop/master/3scale/resources/amp.yml) from /resources folder to your master: e2-user home.

__Note:__ The wildcard domain should be of the format - apps.<kavs/yourid>.sc.osecloud.com

```
scp -i <mykey.pem> /localpath/to/amp.yml ec2-user@PUBLIC IP OF MASTER:~
oc new-app --file /path/to/amp.yml --param WILDCARD_DOMAIN=<WILDCARD_DOMAIN> --param ADMIN_PASSWORD=3scaleUser

```

That is it. Now, we wait for a few minutes for all the 3scale pods to come up.

## Step 5: Verification

To access the 3scale API Manager UI, go to:

https://3scale-admin.apps.kavs.sc.osecloud.com  (use your id)

Login: `admin/3scaleUser`  

Now, you are all set to proceed to 3scale specific labs !

