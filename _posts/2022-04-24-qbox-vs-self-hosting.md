---
layout: post
title: Qbox vs Self-Hosted Elasticsearch
date: 2022-04-24 11:12:00-0400
description: 
tags: elasticsearch data
categories: data
related_posts: false
---

As developers or application owners, we are often faced with a complex decision which one to choose - a managed Elasticsearch provided by professional Elasticsearch-as-a-service companies like Qbox, or our own on-premises or cloud-hosted Elasticsearch clusters using manual installation, configuration, and maintenance. The final decision might depend on numerous factors - cost, upfront or incremental investment in the in-house computer infrastructure, the level of in-house expertise and cost of support, ease of maintenance, scalability concerns, and many more. Thus, the choice between self-hosted and managed Elasticsearch may involve multiple trade-offs which are hard to recognize. In this article, we'll compare these two alternatives: Qbox-hosted and self-hosted Elasticsearch making you aware of salient pros and cons of these two options. 

But before embarking on the comparison, let us spell out basic definitions. 

#### What is Self-Hosted Elasticsearch?

Basically, a self-hosted Elasticsearch may involve two options:

- you are provisioning Elasticsearch on-premises using in-house computer infrastructure and expertise. Your DevOps engineers install, configure, maintain, upgrade and trouble-shoot Elasticsearch cluster/s operated by your company.
- you are running a self-hosted Elasticsearch on the infrastructure of a  Cloud Service Provider (CSP) like Amazon AWS of Google Cloud Platform (GCP). In this case, you also do manual installation and configuration of an Elasticsearch cluster although now using the cloud-based Infrastructure-as-a-Service (IaaS) instead of your in-house resources to provision and scale Elasticsearch clusters. Note that this approach is not the same as using CSP-managed Elasticsearch solutions like Amazon Elasticsearch Service that also make it easy to deploy, secure, operate, and scale Elasticsearch. 

#### What is Qbox-Hosted Elasticsearch? 

With Qbox, you are hosting Elasticsearch remotely having access to the UI and services provided by our company.

-  Qbox automatically installs, provisions, scales, and resizes your  Elasticsearch cluster/s  on remote CSP infrastructure provided by AWS, Rackspace or SoftLayer. 
-  Qbox manages Elasticsearch configuration, monitoring, alerting and provides 24/7 support for your cluster operations.
-  You don't need to interact with CSPs to configure and scale your Elasticsearch. Most things that you need are provided out-of-the-box by Qbox. 

Now, as you have a better understanding of available options, let's compare them in more detail. 

### Installation 

#### **Qbox**

Elasticsearch installation with Qbox is plain and simple. You don't need to download and install any software packages: Qbox will ship the requested Elasticsearch version for you and walk you through a simple installation process at the end of which you'll have a fully operational Elasticsearch cluster on the infrastructure of a CSP you prefer. Qbox currently supports three major CSPs including Amazon AWS, Rackspace, and SoftLayer. 

After you've selected a CSP, Qbox will automatically manage the provisioning of virtual machines and storage for the ES cluster freeing you from the tedious configuration work and the need to create and manage your own accounts on the CSP.  During the installation process, Qbox will also let you specify the number of nodes to create and computing resources (RAM and processors) to service them.  For example, in the image below, we see the Rackspace cluster configuration that offers a wide range of infrastructure option ranges, which allows deploying the exact amount of resources to match your needs. 

![](Image1_new-cluster-rackspace.png)

During the installation process, you can also select the region in which to deploy your ES cluster, Elasticsearch version, and the monitor plug-in.  The entire installation process should not take more than 2 minutes if you are using the Qbox-hosted Elasticsearch. 

#### Self-hosted Elasticsearch

Needless to say, that installing self-hosted Elasticsearch would require much more manual work including finding an Elasticsearch version compatible with your servers, configuring security, linking plug-ins you would like to use with your Elasticsearch, and many more. Another thing is that Elasticsearch installation details differ across operating systems (e.g Windows and Linux). You'll need to spend additional time figuring out the installation process for your platform. Your OS environment should be also prepared. In particular, since Elasticsearch is built using Java, it requires Java environment (at least Java 8). All these major and small prerequisites will make your team spend more time preparing and configuring servers and software to work with Elasticsearch. 

### **Configuration**

#### **Qbox** 

Elasticsearch has good defaults and requires little configuration. However, Qbox abstracts a configuration process even further and simplifies it using an easy-to-use web-based UI. With Qbox-hosted Elasticsearch, you have the most important configuration options available: 

- setting the cluster's name and selecting the cluster's region 
- changing the default number of replicas and shards for your indexes (AWS allows configuring replicas, Softlayer and Rackspace support both)
- configuring traffic rules and security
- choosing plug-ins to install with your Elasticsearch cluster. Qbox supports a wide variety of ES plug-ins like [Elasticsearch Migration](https://github.com/elastic/elasticsearch-migration) , [Graphaware Graph-Aided Search](https://github.com/graphaware/graph-aided-search), [Phonetic Analysis](https://github.com/elasticsearch/elasticsearch-analysis-phonetic), [Combo Analysis](https://github.com/yakaz/elasticsearch-analysis-combo) and many more. 
- enabling alerting, monitoring, and Kibana

**Self-Hosted Elasticsearch**

To configure a self-hosted ES cluster, you may need to work with three files which are `elasticsearch.yml` for configuring Elasticsearch, `jvm.options` for configuring Elasticsearch JVM settings, and  `log4j2.properties` for configuring Elasticsearch logging. Manual configuration can be an error-prone work because you have to enter information by hand and always consult the official documentation to make sure you're doing everything right. 

To illustrate the difference between Qbox and self-hosted ES configuration, let's take a look at setting a cluster's name. All nodes should know this name to join the cluster. As we've seen, in Qbox, you choose the cluster's name during the installation process. In contrast, a self-hosted ES is created with a default cluster name ("Elasticsearch") which should be changed in the configuration file after installation like this: 

```yaml
cluster.name: YOUR CLUSTER NAME
```

Guess what's easier! The answer is obvious: using Qbox UI to configure your cluster without tinkering with configuration files is much easier. 

The same is true of plug-ins. To install a plug-in using a self-hosted ES, you should use the plug-in manager. For example, to install the Phonetic Analysis plug-in, you need to open your terminal and enter the following command:

```sh
sudo bin/elasticsearch-plugin install analysis-phonetic
```

That's easy but what if you need a dozen of different plug-ins which also have to be configured before using them? Also, to work smoothly, plug-ins should be installed on every node in the cluster, and each node should be restarted afterwards. Needless to say, Qbox will take care of this. After you've edited the list of required plug-ins on Qbox, the system will automatically restart each node and re-provision the cluster for you. 

Also, with the self-hosted Elasticsearch, you may need to configure the OS to allow Elasticsearch process to use more system resources than usually allocated by default. That is because, ideally, Elasticsearch should run alone on the server and have access to all resources available to it. 

Finally, Qbox makes it easy for your cluster to go into production since it adjusts the infrastructure and system environments to work at scale. When using self-hosted Elasticsearch, however, you should ensure that the following things are done before going into production: 

- disable swapping
- increase file descriptors
- ensure that sufficient threads are available 
- ensure that sufficient virtual memory is available 
- ensure proper JVM DNS cache settings 

All this makes going into production a challenging task that requires a lot of planning, time, and expertise to be implemented. 

### Scalability 

#### **Qbox**

Scalability is one of the major requirements of the modern distributed applications which is hard to meet using a self-hosted solution. If your unable to cope with the sudden surge of your ES server's load or memory shortage by scaling the underlying infrastructure, you'll probably fail to retain your customers. Qbox solves this problem by enabling automatic resizing of your clusters using AWS, Rackspace or SoftLayer APIs under the hood. To resize your cluster, you don't have to access remote CSP accounts and manually launch and configure new VM instances. Qbox will just allow you to select the size of RAM and a number of replicas, and then automatically adjust the right CPU configuration for your cluster. You'll be able to see how your costs change as well. It can't be easier than that!

![](Image2_resize-cluster-aws.png)

#### **Self-Hosted Elasticsearch** 

Scaling self-hosted Elasticsearch is a complex task that requires a lot of planning and upfront investment in computer infrastructure (e.g servers, memory, network). If your cluster has run out of memory and storage and the company has no additional servers to allocate to your cluster, you'll need to buy some more. That process takes time and costs you money.  After you've obtained new resources, your IT department will need to manually configure and deploy new servers, OS, configure the network, security, and environmental variables for your Elasticsearch cluster. Even if you're using some IaaS provider for hosting the Elasticsearch cluster, you'll still need to manually deploy and configure new VMs for it. 

### Backups, Cloning and Disaster Recovery 

#### **Qbox**

Qbox handles the backup and recovery process for your Elasticsearch cluster automatically. Backups run daily using Elasticsearch's Snapshot API under the hood. Qbox stores these backups on the highly available remote storage during 7 days so you always have access to the recent snapshots of your cluster. Restoring a cluster from the prior backup is plain and simple: just select a backup snapshot you want with '**View backups**' under the "**Manage**" drop-down in your Qbox dashboard and Qbox will re-provision a new cluster using the selected snapshot. 

![](Image3_qbox-backups.png)

This simple backup restoration process via the dashboard allows Qbox customers to quickly take action when issues arise.

In addition to backups, Qbox supports cloning of existing clusters. Using "**Clone from backup**" feature under the "**Manage**" drop-down, you can instantly create a new cluster with the desired setup and configuration. This feature is especially useful when the failure of your cluster is imminent or the cluster is unrecoverable which might happen due to complete resource overload or cloud outage. 

![](Image4_cloning-cluster.png)



#### **Self-Hosted Elasticsearch** 

To create backups with a self-hosted Elasticsearch, you need to use Snapshot and Restore modules manually or develop some sort of back-end service or cron job if you want to create backups and clones automatically. Overall, creating a backup in a self-hosted Elasticsearch involves a lot of requirements:

- in order to clone a multi-node cluster on-premises, you'll need to use a Network File System (NFS) share and make it accessible to all nodes on the same mounting point. 
- after NFS is installed, you'll need to register a repository to save backup snapshots to.
- grant Elasticsearch permissions to write to a new repository
- specify a path to the repository in the ES configuration file
- making a cluster's backup snapshot and testing it 

As you see, backup management in the self-hosted Elasticsearch involves quite a lot of steps. At each step, you should ensure that everything is done correctly because creating snapshots of your data is not something to take lightly. A backup that can't restore your Elasticsearch cluster at a critical time is worth nothing. Therefore, you should think twice before doing Elasticsearch backups manually. Qbox dramatically simplifies this process ensuring that your backups are always available and properly tested to be restored at the most critical time. 

### Kibana

Kibana is an analytics and visualization system designed to work with Elasticsearch. It can be used to search, view and manage your Elasticsearch indices. Kibana allows making sense of huge volumes of data using its simple, browser-based interface. 

#### **Qbox**

Installing Kibana with Qbox is as simple as checking the corresponding box during ES installation process. Qbox will ensure the compatibility of Elasticsearch and Kibana versions, create appropriate credentials for communication between Kibana and Elasticsearch instances, and configure Kibana to access ES by assigning ES host and port. After your Elasticsearch is provisioned, you can access Kibana by clicking the link that may be found in your Qbox dashboard. It's as easy as that! 

![](Image5_Kibana-UI.png)



#### **Self-Hosted Elasticsearch** 

When using a self-hosted Elasticsearch, you'll need to install Kibana manually. It's quite simple but takes extra time. In particular, you should ensure that Kibana and Elasticsearch installations have the same versions.  Moreover, once Kibana is installed, you should configure Elasticsearch host and port numbers, username and password for Kibana to access your cluster. These settings need to be edited in your `kibana.yml` configuration file. At the bare minimum, you'll need to configure `elasticsearch.url` , `elasticsearch.username` and `elasticsearch.password:` in Kibana configuration file. In sum, installing Kibana manually involves quite a lot of steps compared to the automatic installation of Kibana by Qbox. 

### Security

In its basic form, Elasticsearch security settings should ensure the following:

- prevent unauthorized access to your cluster through password protection, 
- enabling IP filtering and role-based access control (RBAC)
- preserving data integrity with SSL/TLS encryption 
- maintaining an audit trail 

Let' see how Qbox and self-hosted Elasticsearch approach these and other security requirements. 

#### **Qbox**

Qbox-hosted Elasticsearch is provisioned with the basic security settings which prevent unauthorized access and ensure data integrity of your cluster. In addition to SSL, all Elasticsearch clusters are provisioned with HTTP Basic authentication that includes username and password. Qbox also provides whitelisting for HTTP and transport traffic, VPC peering, and encryption-at-rest. Qbox currently does not allow custom SSL certificates on the clusters because they are not viable in the long run. 

#### **Self-Hosted Elasticsearch**

To secure your self-hosted Elasticsearch, you'll need to install X-Pack on every node of your cluster. X-Pack is an official Elasticsearch package that supports security, monitoring, alerting, analytics, and more. Although the product comes with a lot of free features, full access to all functionality of the Security (former Shield) module including encrypted communication, RBAC, LDAP and Active Directory authentication, SAML authentication, Audit logging, and more [requires a Platinum subscription](https://www.elastic.co/subscriptions#request-info). In addition to this, X-Pack security module requires a lot of manual configuration that can be only done by qualified computer security professionals.

### Free Alerting and Monitoring 

Think of the use cases for alerting and monitoring in your Elasticsearch application: CPU usage is suddenly surging, response errors are spiking, traffic is decreasing unexpectedly. All these issues tell much about the health and performance of your Elasticsearch cluster and you need to have monitoring and alerting enabled to know about these issues as they arise. 

#### Qbox 

Qbox-hosted ES ships with [ElastAlert](https://github.com/Yelp/elastalert), a simple alerting framework that helps identify anomalies, spikes and other unusual patterns in your Elasticsearch data. You can enable alerting during or after installation of your ES cluster on Qbox. The system will let you define alerting rules in the YAML format to choose scenarios and events when alerts should be sent. In addition, by default, Qbox supports email alerting using Qbox's SMTP host.

![](Image6_ElasAlert.png)

With Qbox-hosted ElastAlert, you can design a customized alerting system that can be used for the following: 

- Link ES alerts to Kibana dashboards
- Create periodic reports based on alerts data 
- Organize your alerts in classes using a unique key field
- Intercept and improve match data

At the moment, ElastAlert supports a variety of alert types including Email, JIRA, HipChat, Slack, Telegram, SNS, and more. 

Also, by default, Qbox supports a number of free monitoring plug-ins like [Elasticsearch Head](https://github.com/mobz/elasticsearch-head),  [Elasticsearch Kopf](https://github.com/lmenezes/elasticsearch-kopf), and [Elasticsearch HQ](https://github.com/ElasticHQ/elasticsearch-HQ) among others. In addition to these plug-ins, Qbox regularly checks clusters for potential issues and notifies users if any problems arise. 

#### Self-Hosted Elasticsearch 

With self-hosted Elasticsearch, you need to select, install and configure alerting and monitoring plug-ins manually. You should also configure the alerting transport such as your email service or Slack channel. ElastAlert is a great choice for these purposes but your team will have to spend extra time to integrate it. Other great alerting options for self-hosted ES include X-Pack that supports a variety of alerting types, integration with your monitoring infrastructures, and more. However, X-Pack alerting (via Watcher) requires a Gold subscription. 

### **Support**

Support is critical when something unexpected happens. And it always happens in complex environments like Elasticsearch clusters. Having a professional support at your fingertips to instantly address the issues is one of the major requirements for successfully running your Elasticsearch cluster.

#### **Qbox**

At Qbox,  free support is a part of the support response [Service Level Agreement (SLA)](https://qbox.io/support). It covers one hour a day for production issues, four hours for product questions, eight hours for updates, migrations and maintenance, and 24 hours for general issues. Qbox's dedicated support includes certified database and ES professionals who have a deep knowledge of this technology and resources to address dozens of reasons why your ES node might be unresponsive. Having a dedicated support at your fingertips is even more important if you're running Elasticsearch in production since you might not have enough time and expertise to fix emerging issues. 

#### **Self-Hosted Elasticsearch**

To run Elasticsearch on-premises, your company needs to hire DevOps specialists for server maintenance, management, troubleshooting and making upgrades. A qualified specialist/s should be always available to fix issues are they arise. Needless to say, hiring such specialists introduces additional costs. Also, many companies have a misconception about Elasticsearch support services offered by major CSPs. Many people expect that when self-hosting Elasticsearch on the CSP infrastructure (e.g Amazon AWS) and the cluster is non-responsive for some reason, they can reach support to fix the issue. The problem is that, however, issues in your Elasticsearch are frequently not at the node level while the CSP's responsibility for you is only to ensure that your VMs are up. Remember that Elasticsearch is an open-source technology created by the community and having a huge code base. You should not expect CSPs to have the same level of expertise in Elasticsearch as in their proprietary products. 

### Summary 

We're almost there as we've covered key differences between Qbox and self-hosted Elasticsearch. Now, hopefully, you have a better understanding of what both solutions bring to the table. Let's briefly sum up their key benefits and pitfalls. 

#### Qbox Benefits

- Easy Elasticsearch installation, configuration, and management using a simple web-based UI 
- dedicated provisioning of computer resources for your cluster
- one-click scaling and resizing of your Elasticsearch cluster in a fraction of time without query interruption
- immediate and continuous disaster recovery with Qbox's  high-availability storage system mirrored in the cloud
- built-in security and monitoring of your cluster's health. 
- free alerting and other useful plug-ins out-of-the-box
- an efficient support response Service Level Agreement (SLA)

**Self-Hosted ES Benefits **

- More fine-grained control over configuration. You can have a better control over how ES works with a full access to ES and Kibana configuration files and Elasticsearch API.
- You can add proprietary products like X-Pack not supported by Qbox to your Elasticsearch installation. With an option to select any plug-ins and services not supported by Qbox, you can design any customized Elasticsearch stack that fits your needs. 

Hosting Elasticsearch on-premises and having more fine-grained control over configuration, however, comes with certain limitations:

- Requires substantial upfront investment in computer infrastructure and software
- Necessitates  in-house expertise or paid third-party support for  server maintenance and management of updates and backups.
- Scaling self-hosted Elasticsearch might take more time due to the manual deployment of new servers and configuring them. 

### **Conclusion**

Self-hosted Elasticsearch is definitely an option if you want a highly customized ES solution and have enough in-house expertise and resources to support maintenance, recovery management, scaling, and upgrading of your Elasticsearch clusters. Hosting ES on-premises, however, will inevitably result in more planning and upfront investment and may turn out to be unsustainable in the long run if capacities are overestimated. The lesson learned by many companies who opted for self-hosted Elasticsearch is that if you can find an expert offering a service you need - you better use them. Although it might be cheaper in the short term to manage Elasticsearch yourself, managing it at scale might end up in a lot of bottlenecks, maintenance issues, bugs all of which means interruptions of your service. There are simply dozens of things that one need to consider when using Elasticsearch in production. These issues can be professionally addressed by someone who specializes in providing Elasticsearch-as-a-service as its major business. Qbox experts will take the pain of building and managing your Elasticsearch cluster away from you which will ultimately free up time to concentrate on building excellent products letting others take care of the rest. 

