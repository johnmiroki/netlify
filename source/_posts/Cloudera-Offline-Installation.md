title: Cloudera Offline Installation
author: John Miroki
date: 2018-08-17 17:15:28
tags:
---
This is a listing of all the work needed for installing CM and CDH5 on CentOS 7 for study purpose. All servers are virtually hosted on VMware Workstation. 

A few words before I begin. I find the official documents both informative and confusing at the same time, partially owing to the fact that there are 
several paths to installation and these paths diverge at one time and overlapse at another time. One gotta makes sure to stay within the chosen path, otherwise one may end up tripping down the rabbit hole. Some other confusion results from lacking a big picture of different roles played by vairious components/softwares. Diving in the details of the documents proves not the best choice I've made. Anyway, if ever I install the system again, the first two documents I should read are: 
* https://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_ig_intro_to_cm_install.html
* https://www.cloudera.com/documentation/enterprise/5-5-x/topics/installation_installation.html#concept_qpf_2d2_2p

TL;DR:
The path I choose is to install Cloudera Manager first, and let it install other components automatically for me. At the same time, I prefer offline repositories for all dependencies. Also, between a "Demonstration and proof of concept deployments" and a "Production deployments" [ref](https://www.cloudera.com/documentation/enterprise/5-5-x/topics/installation_installation.html#concept_qpf_2d2_2p), I prefer the latter.



## Preparetion

### SSH Setup
To facilitate fast deployment, master host should be able to get access to other hosts with ssh, password free:
* `ssh-keygen -t rsa`
* `ssh-copy-id cluster0[2-4]`

### Networks
I mainly use `nmtui` tool to configure the network.

let the hosts file speaks
```hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.146.101 cluster01.john.how cluster01
192.168.146.102 cluster02.john.how cluster02
192.168.146.103 cluster03.john.how cluster03
192.168.146.104 cluster04.john.how cluster04
```

make sure all hosts have a copy:
```shell
scp /etc/hosts @cluster02:/etc/
```

### Security & Firewall
shut 'em down. 
* shutdown firewall (or iptables)
```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
```
* modify `/etc/selinux/config` -> `SELINUX=disabled` (reboot needed)

### Repositories
Since all servers cannot connect to the internet (when they do, it's slow anyway), I'm going with the offline route.

#### Parcels

Accoding to the official [documents](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_custom_installation.html#cmig_topic_21), "For a parcel installation, only the Cloudera Manager Server needs Internet access, but for a package installation, all cluster hosts require access to the Cloudera repository". Although my master server has internet access, I'm going to set up an internal parcel repository anyway, so that I can utilize some downloader software to expedite the downloading process.

You can find the detailed steps [here](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_create_local_parcel_repo.html#cmig_topic_21_5).

The abreviated steps are :

1. Set up an Apache Http Server
```shell
yum install httpd
service httpd start
```
2. Download the parcel files for CentOS from https://archive.cloudera.com/cdh5/parcels/5.15/
These are the files I think relavent for me
	* CDH-5.15.0-1.cdh5.15.0.p0.21-el7.parcel
    * CDH-5.15.0-1.cdh5.15.0.p0.21-el7.parcel.sha1
    * manifest.json
3. Move files to server directory:
```shell
mkdir -p /var/www/html/cloudera-parcels/cdh5/<cdh5_version>/
mv *.parcel* /var/www/html/cloudera-parcels/cdh5/<cdh5_version>/
mv manifest.json /var/www/html/cloudera-parcels/cdh5/
chmod -R ugo+rX /var/www/html/cloudera-parcels/cdh5/<cdh5_version>/
```
4. Verify the server by browsing `http://<web_server>/cdh5/<cdh5_version>`

#### Package

I am not clear at the moment that if both Parcel and Package are needed, but since install the Oracle Java JDK needs the package, I'm going to set up the offline repository (Since the requested http server has been established, I'm going to skip those steps)
1. Download the tarball: `https://archive.cloudera.com/cm5/repo-as-tarball`
2. `tar xvfz cm5.15.1-centos7.tar.gz`
3. `mv cm /var/www/html`
4. `chmod -R ugo+rX /var/www/html/cm`
5. To verify, navigate to `http://hostname:port/cm`
6. Config yum repo file: `vi /etc/yum.repos.d/myrepo.repo`
```
[myrepo]
name=myrepo
baseurl=http://cluster01.localdomain/cm/5.15.1
enabled=1
gpgcheck=0
```
copy to every host: `scp /etc/yum.repos.d/myrepo.repo cluster02:/etc/yum.repos.d/`

### MySQL
This part has been lost... Reference [here](https://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_ig_mysql.html#cmig_topic_5_5_2)
One thing to note is one should NOT use the lastest version, since JDK 7 will be installed (See below) by default and the latest connector requires JRE 1.8.

### Java JDK
`yum install oracle-j2sdk1.7`

## Installation

Having jumped all the loops and hoops, I can finally get down to the real business: 

### Install the Cloudera Manager Server Packages
`yum install cloudera-manager-daemons cloudera-manager-server`

### Preparing a Cloudera Manager Server External Database
`/usr/share/cmf/schema/scm_prepare_database.sh mysql [-uuser -p] scm scm password`

### Start the Cloudera Manager Server
`service cloudera-scm-server start`

### Start and Log into the Cloudera Manager Admin Console
1. Look for errors `tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log`
2. Navigate to `http://Server host:7180`
3. Login admin admin

### Installation Process
1. After a few Agreements and edition selection, I'm asked to choose the hosts. And I input `cluster0[1-3]`, leaving cluster04 out of it (saving it for future)
2. When it comes to parcels, select "more options" and input the local parcels server address `http://192.168.146.101/cloudera-parcels/cdh5/5.15.1/` (possibly need to move menifest.json to this folder)
3. A bunch of self-explainary options
4. The critical step is to tell the installer to use your own repository:`http://192.168.146.101/cm/5.15.1/`

## Troubleshooting

* Nodemananger start failure: run this command on every Nodemanager host `chmod -R 755 /var/lib/hadoop-yarn/`

## Reference
* The Official [](https://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_ig_install_path_b_auto.html#id_bwc_4ck_pt)
* The Official [Before You Install](https://www.cloudera.com/documentation/enterprise/latest/topics/installation_reqts.html#pre-install)
* Random offline installation guide (https://www.zybuluo.com/sasaki/note/242142)
* not directly related to installation per se, but gives you a general idea of how many servers needed and what roles they each bear [Recommended Cluster Hosts and Role Distribution](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_host_allocations.html#host_role_assignments)
* 
* 


`sudo yum --nogpgcheck localinstall oracle-j2sdk1.7-1.7.0+update67-1.x86_64.rpm`