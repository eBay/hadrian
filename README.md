## Archived Project
Please note that this project is archived and read-only.

#hadrian

A set of utilities used to build a Hadoop cluster with management tools.  The overall goal of this project is to allow rapid spin up of a variety of big data distributed systems on either physical or virtual platforms.

* CDH3
* CDH4
* more distributions to follow.  Please see the wiki roadmap (coming soon).

##Features
Hadrian was created to rapidly install and deploy a Cloudera Hadoop cluster onto both Openstack Virtual machines and physical machines. 
The project has been evolving though, so a short list of new features follows:
- Hadrian can also deploy to existing machines.  A user needs sudo permissions (password or passwordless) to install software
- OS Tuning has been included.  SELinux is disabled, iptables are disabled, swappiness is tuned, overcommit_memory is also tuned.  more to come
- Hadrian supports the following Haodop Components, HDFS/Mapreduce, HBase, Zookeeper, Hive, Pig and Mahout
- Cloudera Manager 4.7.x+ is also supported and deployed to manage the Hadoop instances.

- NOTE: Only RHEL/Centos 6.0 is supported at this time.  This will NOT work on Ubuntu/Debian variant machines.

##Getting Started
1. You'll need a machine with Python ~2.6, preferably with internet access and access to the machines you are planning to deploy.
2. You'll need to install Python PIP for easy package management.  If you don't have internet access, don't worry.  RPM creation directions are below.
3. Install a C compiler on your master machine.  On linux, sudo yum install gcc-c++ -y
4. Install PIP.  Website here: https://pypi.python.org/pypi/pip
5. Post pip install, you will need to pip install the following
- sudo pip install Fabric (greater than 1.6)
- sudo pip install setuptools
- sudo pip install argparse
- sudo pip install cm_api
At the moment, you need to have the novaclient even if you aren't using openstack.  this will be fixed in the near future.
- sudo pip install novaclient

these are the just-in-case-they-are-needed packages
- sudo pip install json
- sudo pip install simplejson

## No internet? No worries.
If you don't have internet on your soon to be cluster, things get a little trickier.  It's not insurmountable though.

You need the following:

- python setuptools installed (you can use easy_install or pip)
- yum install rpm-build
- yum install gcc-c++
- yum install python-devel

Once you have those, then things get a little bit easier.  Depending on what packages, you might need some other RPMs but these are the major packages.

Steps
1. Download the tarball of your package, in this example, argparse-1.2.1
2. tar xf argparse-1.2.1.tar.gz
3. cd argparse-1.2.1
4. python setup.py bdist --format=rpm
5. watch a bunch of stuff get written to the screen
6. cd ./dist
7. enjoy your shiny new RPMs.    

##Configuring Hadrian

###conf/hadrian.ini
This is one of the primary configuration files for Hadrian.  We'll go through each block and explain what's what and whether or not you need to modify the configuration.

####Globals

```
[Globals]
#This can be named anything, easier to avoid spaces though
cm.cluster.name=hadrian-cluster
cm.username=admin
cm.password=admin
cm.port=7180
cdh.cluster.version=cdh3

#tell hadrian how you want to push CDH software.  Parcels or RPMs.
# value should be: parcels or rpms
cdh.distribution.method=rpms

# If you are using Parcels, you need to set this: 
cdh.parcel.version=4.1.4-1.cdh4.1.4.p110.11

#Database Configuration Section
#Choose a database backend for hive/CM Options are postgresql or mysql
# if you choose postgresql, it will be installed. if you choose mysql, then you will need an existing mysql server

#Options postgresql or mysql
system.database=postgresql

```
The Globals section.  At the top, you can see some basic Cloudera Manager information.  
* _cm.cluster.name:_ This is the name of you clusters in Cloudera Manager.  It's also used for configuration mappings.  See below for more details.
* _cm.username:_ This should be left as admin for new clusters.
* _cm.password:_ This should be left as admin for new clusters.
* _cm.port:_ This should be left as 7180 for new clusters.
* _cdh.cluster.version:_ Right now, you have two options, cdh3 or cdh4
* _cdh.distribution.method:_ Again, two options.  rpms or parcels.  This is the format that CDH softare is installed.
* _cdh.parcel.version:_ This is only required if you choose parcels as your distribution method.
* _system.database:_ There are two options, mysql and postgresql.  If you choose mysql, you will need to install and do initial configuration of your MySQL database outside of Hadrian.  You will also need your mysql root password for creating users, doing grants.  For postgresql, Hadrian will install it for you.

####Database Information
```
[postgresql]
db.port=5432

[mysql]
db.host=mysql_db_fqdn
db.port=3306

[DB Users]
#This section is where passwords are entered for the database users/schemas to be created on the database server.
#The asssumption is that you have permissions to create the users.
#TODO add option to use exsting schemas
cm.db.password=scmsecure
amon.db.password=amonsecure
smon.db.password=smonsecure
rman.db.password=rmansecure
hmon.db.password=hmonsecure
metastore.db.password=metastoresecure
```

####postgresql section
* _db.port:_ this is the default port for postgres.  Just leave it as 5432

####mysql

* _db.host:_ This is your MySQL server's hostname.  Use the fullly qualified domain name.
* _db.port:_ This is your MySQL servers database port. Configure accordingly. 

####DB Users
Set your desired password for the various Cloudera Manager database users.  The last entry is your Hive Metastore's entry.  Even if you aren't using Hive, this schema/user will be created.  You must have an entry for every database schema/user

```
[OpenStack Information]
ostack.os.image.name=nova_image_name
ostack.auth.url=https://openstack_auth_server:5443/v2.0
host.domain=openstack_hosts_domain
```
####Openstack Information
* Openstack is semi-supported at this time.  You can deploy to Openstack VMs, but automatic creation hasn't been working very well lately on our environments.  Further development (and environment stabilization) is required.

```
[JVM Information]
jdk.download.url=http://archive.cloudera.com/cm4/redhat/6/x86_64/cm/4/RPMS/x86_64/jdk-6u31-linux-amd64.rpm
jdk.home.dir=/usr/java/jdk1.6.0_31
```
####JVM Information
* _jdk.download.url:_ This is the JVM that you wish to have on your cluster.  This will be use by Hadoop and Cloudera Manager.  You should set this to either the Cloudera hosted JVM URL (above) or an internally hosted JVM.  As will all things Hadoop, Oracle JDKs are recommended above all others.

```
[Yum Repo Information]
# URL to the Cloudera Manager Yum Repository File
cm.yum.repo.file=http://archive.cloudera.com/cm4/redhat/6/x86_64/cm/cloudera-manager.repo

# URL to the CDH Yum Repository File
cdh.yum.repo.file=http://archive.cloudera.com/cdh4/redhat/6/x86_64/cdh/cloudera-cdh4.repo
```
####Yum Repo Information
* _cm.yum.repo.file:_ This is the Cloudera Manager yum repository file.  NOTE: the one above takes the latest and greatest Cloudera Manager.  If you need a specific version, then you will need that hosted on a web server in a similar manner.
* _cdh.yum.repo.file:_ This is the repository file for Cloudera Hadoop.  The above example is for CDH4.  If you have a specific version, then you will need to set up your own repo file on a webserver.  NOTE: This is only used for RPM deployments.  If you choose parcels, this URL doesn't matter.

###conf/cluster_specs.ini

#### Top Level Cluster Information Section
```
[hadrian-cluster]
description=This is your hadrian cluster information.  
```
* This is unused at the moment, this section maps directly to your _cm.cluster.name_ value in _hadrian.ini_.  They MUST match.

#### Enterprise/Master Node Section
```
[hadrian-cluster-en]
# This is a comma separated list of all of your master/enterprise nodes.  FQDNs please
full.list=hadrian-en1.dev.ebay.com,hadrian-en2.dev.ebay.com,hadrian-en3.dev.ebay.com,hadrian-en4.dev.ebay.com

# This is the Cloudera Manager server
cm.server=hadrian-en1.dev.ebay.com

# For HDFS High Availability in CDH4, this should be a comma seperated list of two servers
name.node=hadrian-en2.dev.ebay.com

# For Non-HDFS HA, this is your Secondary Name Node host
secondary.namenode=hadrian-en3.dev.ebay.com

# Only used for CDH4 HDFS HA. You must have 3 hosts.
journal.node=hadrian-en2.dev.ebay.com,hadrian-en3.dev.ebay.com,hadrian-en4.dev.ebay.com

# The CDH4 WebHDFS server
httpfs=hadrian-en4.dev.ebay.com

# The MRv1 Job Tracker host.
job.tracker=hadrian-en4.dev.ebay.com

#HBase Mater hosts, comma separated list of FQDNs
hbase.masters=hadrian-en3.dev.ebay.com,hadrian-en4.dev.ebay.com

#Zookeeper Hosts.  Odd Numbers please, 1, 3, or 5 hosts, comma separated FQDNs/
#NOTE: Zookeeper is also used for HDFS HA regardless of whether or not you use HBase
zookeepers=hadrian-en1.dev.ebay.com,hadrian-en2.dev.ebay.com,hadrian-en3.dev.ebay.com

# Hive Metastore host. Comma Seperated FQDNs please.
# NOTE: at the moment, this must be the CM server for JDBC reasons.  Still working out bugs in the Hive deployment
hive.metastores=hadrian-en1.dev.ebay.com
```
* Descriptions are in the configuration above.  Some lines are only necessary for CDH4 deployments.  
* _NOTE:_ the most important configuration to be made, other than setting the hosts, is naming the section.  This should be <cm.cluster.name>-en.  If you don't match it exactly for your cluster name, then Hadrian won't work.


####Cluster Data Node Section
```
[hadrian-cluster-dn]
rack1=hadrian-dn1.dev.ebay.com,hadrian-dn2.dev.ebay.com
rack2=hadrian-dn3.dev.ebay.com,hadrian-dn4.dev.ebay.com
```
* _Section Name:_ Again, this should map to your cm.cluster.name in _hadrian.ini_. e.g. _cm.cluster.name-dn_
* _rackN:_ Hadrian support Hadoop Rack Awareness by setting the rack locations in Cloudera Manager.  The key rackN, should be the name of the rack.  The Values are a comma separated list of Data Nodes FQDNs that are in the Rack.  
* If you don't have multiple racks of servers, or just don't care, just set this to be rack1=your Data Node list


###conf/cm.cluster.name folder

* This folder contains the configuration files used by Hadrian to set up your Hadoop Cluster. This should have all the basic configurations that you need to run a small scale (single developer sized) cluster.  Larger installations should review the configurations and set accordingly.

####Example Directory

* ./conf/hadrian-cluster
* ./conf/hadrian-cluster/hdfs_config.ini
* ./conf/hadrian-cluster/mapreduce_config.ini
* ./conf/hadrian-cluster/hbase_config.ini
* ./conf/hadrian-cluster/zookeeper_config.ini
* ./conf/hadrian-cluster/hive_config.ini
* ./conf/hadrian-cluster/oozie_config.ini NOTE: not supported at this time

####Example Configuration File - CDH4 HDFS 
```
######################################################################################################
# HDFS Configurations
# NOTE: list all configurations that you want to apply in a comma separated list called config groups
# If you want to create additional Role Config Groups, simply create a section below with the configs
# and add the section name to the config_groups list 
######################################################################################################

[HDFS]
config_groups=hdfs1-NAMENODE-BASE,hdfs1-DATANODE-BASE,hdfs1-FAILOVERCONTROLLER-BASE,hdfs1-HTTPFS-BASE,hdfs1-HTTPFS-BASE,hdfs1-GATEWAY-BASE,hdfs1-BALANCER-BASE,hdfs1-SECONDARYNAMENODE-BASE,hdfs1-JOURNALNODE-BASE

[hdfs1-svc-config]
dfs_replication=3
zookeeper_service=zookeeper1
dfs_namenode_quorum_journal_name=journal2hdfs1

[hdfs1-NAMENODE-BASE]
dfs_name_dir_list=/x/cdh/hdfs/nn
namenode_java_heapsize=134217728
namenode_log_dir=/x/cdh/log/hadoop/nn
dfs_federation_namenode_nameservice=openstratus-ns
autofailover_enabled=false

[hdfs1-SECONDARYNAMENODE-BASE]
fs_checkpoint_dir_list=/x/cdh/hdfs/snn
dfs_secondarynamenode_nameservice=openstratus-ns
secondary_namenode_java_heapsize=134217728
secondarynamenode_log_dir=/x/cdh/log/hadoop/snn

[hdfs1-DATANODE-BASE]
dfs_data_dir_list=/x/cdh/hdfs/dn
datanode_java_heapsize=134217728
dfs_datanode_du_reserved=104857600
datanode_log_dir=/x/cdh/log/hadoop/dn

[hdfs1-GATEWAY-BASE]
dfs_client_use_trash=false

[hdfs1-FAILOVERCONTROLLER-BASE]


[hdfs1-HTTPFS-BASE]

[hdfs1-BALANCER-BASE]

[hdfs1-JOURNALNODE-BASE]
dfs_journalnode_edits_dir=/x/cdh/hdfs/jn
```
Okay, here's how this works.  At the top, there's the section HDFS.  That is a mapping to the Cloudera Manager Service Type.

* config_groups=comma separated list of section names

When Hadrian runs, it sets up the various services (HDFS, Mapreduce, etc).  Hadrian grabs the config_groups list, splits it, then applies the configurations to the service's role configuration groups.  If this doesn't make sense, just leave that list alone.  It will become clear when you are more familiar with Cloudera Manager.

The ONLY exception to that rule is the hdfs1-svc-config.  This section contains the Service level configurations.

For the most part, new users won't need to alter this much.  Maybe change the paths, etc.  Memory settings are in bytes.

NOTE: If you need/want to add multiline entries, that's totally fine, just fix the indenting.  See the example below.

```
[mapreduce1-JOBTRACKER-BASE]
jobtracker_mapred_local_dir_list=/x/cdh/mapred/jt/local
jobtracker_log_dir=/x/cdh/log/hadoop/jt
mapred_job_tracker_handler_count=40
jobtracker_java_heapsize=134217728
mapred_jobtracker_taskScheduler=org.apache.hadoop.mapred.FairScheduler
mapred_fairscheduler_poolnameproperty=mapred.job.queue.name
mapred_queue_names_list=default,dev
mapred_fairscheduler_allocation=<?xml version="1.0"?>
                                <allocations>
                                    <pool name="dev">
                                        <minMaps>4</minMaps>
                                        <minReduces>2</minReduces>
                                    </pool>
                                    <pool name="default">
                                        <minMaps>2</minMaps>
                                        <minReduces>1</minReduces>
                                    </pool>
                                </allocations>

```

### conf/cloudera-manager
Cloudera Manager also has some configurations that can be modified.  The cm.ini file is slightly different than the cluster configurations.  For one, the defaults are in a section below for easy reference.  To overide, copy and paste the overrid and values in the top section.  See the two examples below.
```
[cloudera-manager-updates]
REMOTE_PARCEL_REPO_URLS=http://repo1.dev.ebay.com/altrepo/cdh/parcel-4.1.4-p110.11/,http://repo2.dev.ebay.com/altrepo/cdh/parcel-4.1.4/lzo-parcel/
PARCEL_DISTRIBUTE_RATE_LIMIT_KBS_PER_SECOND=51200

[cloudera-manager-defaults]
SYSTEM_IDENTIFIER=default
PARCEL_PROXY_PASSWORD=None
MANAGES_PARCELS=true
DOWNLOAD_PARCELS_AUTOMATICALLY=false
ENABLE_API_DEBUG=false
EVENTS_WIDGET_SEARCH_ON_LOAD=true
LDAP_URL=None
PARCEL_USERS_GROUPS_PERMISSIONS=true
PARCEL_REPO_PATH=/opt/cloudera/parcel-repo
PARCEL_PROXY_PORT=None
PARCEL_AUTODOWNLOAD_PRODUCTS=CDH
LDAP_BIND_DN=None
TSQUERY_STREAMS_LIMIT=250
CLUSTER_STATS_PATH=None
MISSED_HB_CONCERNING=5
PARCEL_UPDATE_FREQ=60
HTTP_PORT=7180
WEB_TLS=false
CM_HOST_NAME=None
TRUSTSTORE_PASSWORD=None
PHONE_HOME=true
USING_HELP_FROM_CCP=true
SESSION_REMEMBER_ME=true
PARCEL_PROXY_PROTOCOL=HTTP
HEARTBEAT_INTERVAL=15
NEED_AGENT_VALIDATION=false
TRUSTSTORE_PATH=None
LDAP_GROUP_SEARCH_FILTER=None
KEYSTORE_PASSWORD=None
LDAP_USER_SEARCH_BASE=None
CUSTOM_BANNER_HTML=None
AGENT_PORT=7182
COMMAND_STORAGE_PATH=/var/lib/cloudera-scm-server
LDAP_USER_GROUPS=
PARCEL_PROXY_SERVER=None
CUSTOM_HEADER_COLOR=BLACK
LDAP_BIND_PW=None
LDAP_TYPE=ACTIVE_DIRECTORY
LDAP_GROUP_SEARCH_BASE=None
DISTRIBUTE_PARCELS_AUTOMATICALLY=false
LDAP_DN_PATTERN=None
CLUSTER_STATS_COUNT=10
ALLOW_USAGE_DATA=true
SECURITY_REALM=HADOOP.COM
PARCEL_MAX_UPLOAD=25
LDAP_USER_SEARCH_FILTER=None
CUSTOM_IA_POLICY=None
GEN_KEYTAB_SCRIPT=None
KEYSTORE_PATH=None
PARCEL_PROXY_USER=None
CLUSTER_STATS_DEFAULT_SIZE_MB=100
HTTPS_PORT=7183
MISSED_HB_BAD=10
AUTH_SCRIPT=
AGENT_TLS=false
REMOTE_PARCEL_REPO_URLS=http://archive.cloudera.com/cdh4/parcels/latest/,http://archive.cloudera.com/impala/parcels/latest/,http://beta.cloudera.com/search/parcels/latest/
COMMAND_EVICTION_AGE_HOURS=17520
CLUSTER_STATS_SCHEDULE=WEEKLY
CLUSTER_STATS_TMP_PATH=None
PARCEL_SYMLINKS=true
SESSION_TIMEOUT=1800
CLUSTER_STATS_START=10/22/2012 5:50
NT_DOMAIN=None
PARCEL_DISTRIBUTE_RATE_LIMIT_KBS_PER_SECOND=51200
AUTH_BACKEND_ORDER=DB_ONLY
HEARTBEAT_LOGGING_DIR=None
LDAP_ADMIN_GROUPS=
```

###conf/postgresql

These are the configuration files for your postgresql server.  Unless you know what you are doing, best to leave these alone.  


###conf/mysql

This directory contains a single file, db-setup.sql.  This is used to create the users/databases on your MySQL server.  This should not be altered.



##Final Prep Work

All right, you've been patient and read through all that.  If you just want to get moving, then you probably skipped a lot of that noise above.  There's a couple things you should check before you start.

### Prep work
1. Make sure that all of your hosts know EXACTLY who they are.  This means, that when you login to your boxes, you can type the following and get back proper responses.
* hostname (returns the hostname, preferably FQDN)
* hostname -f (returns the FQDN of the host)
* hostname -i (returns a real IP. NOT 127.0.0.1)

If you find your boxes not doing the above, then you need to fix this.  It will break Hadrian and your Hadoop cluster will NOT be happy.  Just make sure you can fix it.  See the Wiki for help -add wiki link-

2. Make sure you have some form of full root or sudo.
* ssh to your host as root (with a password or passwordless, Hadrian doesn't care.)
* ssh to your host as you, sudo -l (should return all.  Again, with or without a password, Hadrian doesn't care.)

3. You've installed the python packages listed above on a host of our choice.  This is where you will be running Hadrian from, so you should have the Hadrian Software installed.


## FINALLY! Time to Run Hadrian
The moment of truth. You've ventured through documentation, fought with configs, and now your labors near completion.

###Installation
1. cd into your hadrian directory on the host that you installed all those Python packages.
2. ./hadrian.sh
3. Answer the questions as best you can.  If you don't need a password for your root user, then just hit enter.
4. Sit back and watch it go. 


###FAILURE or I'm just tired of my cluster.

* If you had a problem on your installation, then never fear, you can clean your install fairly easily.  
* Clean is pretty dumb at this point.  It's a sledgehammer that needs to be turned into a scalpel.  Treat it as such.
* NOTE: For MySQL, you'll need to clean the DB yourself.  
* SECOND NOTE: This is really important.  If you have running services in Cloudera Manager, you MUST stop them before you run clean. Otherwise you end up with a mess and you need to go around killing the CM supervisord process.  yeah. it sucks, don't miss this step.

1. cd into your hadrian directory
2. ./clean.sh
3. Answer the questions.  The only ones that really matter are the user and password entries.  The rest don't matter.  
