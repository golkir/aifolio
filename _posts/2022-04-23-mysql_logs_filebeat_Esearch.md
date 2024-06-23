---
layout: post
title: How to ship MySQL Logs to Elasticsearch with Filebeat
date: 2022-04-23 11:12:00-0400
description: 
tags: elasticsearch databases
categories: sample-posts
related_posts: false
---

Effective log management involves a possibility to instantly draw useful insights from millions of log entries, identify issues as they arise, and visualize/communicate patterns that emerge out of your application logs. Fortunately, ELK stack (Elasticsearch, Logstash, and Kibana) makes it easy to ship logs from your application to ES collections for storage and analysis. Recently, Elastic infrastructure was extended by useful tools for shipping logs called Beats. Filebeat is a part of Beats tool set that can be configured to send log events either to Logstash (and from there to Elasticsearch), or even directly to the Elasticsearch. The tool turns your logs into searchable and filterable ES documents with fields and properties that can be easily visualized and analyzed. 

In the previous [post](https://qbox.io/blog/how-to-ship-linux-system-logs-elasticsearch-filebeat), we have already discussed how to use Filebeat to ship Linux system logs. Now, it's time to show how to ship logs from your MySQL database via Filebeat transport to your Elasticsearch cluster. Making MySQL general and slow logs accessible via Kibana and Logstash will radically improve your database management, log analysis and pattern discovery leveraging the full potential of ELK stack. 

### Tutorial

To complete the tutorial, we are using:

1. An Ubuntu 16.04 LTS distribution.

2. A Qbox hosted Elasticsearch Cluster with a Kibana support.  

   Qbox.io Elasticsearch clusters outperform self-hosted ES solutions thanks to the platform's easy scalability, 24/7 uptime, easy integration with Logstash and Kibana, and free support.

   For this post, we will be using hosted Elasticsearch on Qbox.io. You can sign up or [launch your cluster here](https://qbox.io/signup?utm_source=blog&utm_campaign=tutorial&utm_term=launch_your_cluster&utm_medium=article), or click "Get Started" in the header navigation. If you need help setting up, refer to [Provisioning a Qbox Elasticsearch Cluster](https://qbox.io/blog/provisioning-a-qbox-elasticsearch-cluster?utm_source=tutorial&utm_term=provision&utm_medium=article&utm_campaign=index-attachments-files-elasticsearch-mapper)

3. MySQL 5.7

4. Filebeat 5.6.2

### **Installing and Configuring MySQL**

If you haven't installed MySQL yet, you can use these steps to install and configure it. The first thing you need to do is to update your system.

```sh
sudo apt-get update
```

And then install MySQL like this:

```sh
sudo apt-get install mysql-server
```

During the installation, you will be prompted to set your root password. Make note of it because it will be required to manage your MySQL database.

The next step is configuring MySQL to write general and slow query log files since these configurations are disabled by default.  To change configurations, you will need to edit `my.cnf` file that contains user database settings. 

```sh
sudo nano /etc/mysql/my.cnf
```

A working configuration for general and slow queries should look like this:

```assembly
general_log      = 1
general_log_file = /var/log/mysql/mysql.log
slow_query_log   = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 1
log_queries_not_using_indexes = 1
```

Please, note that in MySQL versions < 5.1.29 the variable `log_slow_queries` was used instead of `slow_query_log`. 

Make sure to restart MySQL after making these changes:

```sh
sudo service mysql restart
```

Now, your MySQL is ready for writing slow queries that will be shipped to your Elasticsearch cluster via Filebeat.

### Installing and Configuring Filebeat** 

The next step is installing Filebeat on our Ubuntu 16.04 machine. For this tutorial, we are using the latest version of Filebeat (5.6.2) released on September 26, 2017. Once Filebeat is installed, it will be working as an agent that is sending all MySQL logs to Elasticsearch or Logstash for storage and processing. You can install Filebeat from RPM, DEB package, or source. The latest version of Filebeat [can be downloaded here.](https://www.elastic.co/downloads/beats/filebeat)

After we have installed Filebeat, we need to configure it to work with MySQL logs in our system. Similarly to other software on Linux, the default configuration of Filebeat is stored inside `/etc/filebeat` directory. There you will find `filebeat.yaml` configuration file with some examples of Filebeat configurations. Backup this file in case if something goes wrong and create a new `filebeat.yaml` file for our MySQL configuration.  

The initial configuration file for Filebeat will have the following content:

```yaml
filebeat.prospectors:
- input_type: log
  paths:
    - /var/log/mysql/*.log
  registry: /var/lib/filebeat/registry
output.elasticsearch:
        hosts: ["YOUR ELASTICSEARCH HOST"]
        protocol: "https"
        username: "YOUR USERNAME"
        password: "YOUR PASSWORD"
```

After editing the configuration file, don't forget to restart Filebeat

```sh
sudo service filebeat restart
```

The above configuration settings do several things:

1. Filebeat prospectors define the type of logs shipped and their location.
2. Output tells the Filebeat the location where log messages should be sent (i.e Qbox hosted Elasticsearch cluster in our case)

Also, as you can see, Filebeat supports wildcard entries for path settings. So, instead of specifying all log files to be watched, we just define a wildcard location `/var/log/mysql/*.log` which tells Filebeat to watch all files in the MySQL log directory. 

In the configuration file, we have also specified the location of **Filebeat registry file**. The registry file is very important because it stores the state and location information that Filebeat uses to track upcoming logs. This file prevents Filebeat from sending the same entries all over again.  Note the registry setting in our example configuration file above, `registry: /var/lib/filebeat/registry`.

```javascript
{"source":"/var/log/mysql/mysql-slow.log","offset":364,"FileStateOS":{"inode":1454517,"device":2049},"timestamp":"2017-09-27T17:43:06.877125109+03:00","ttl":-1},{"source":"/var/log/mysql/mysql.log","offset":5406,"FileStateOS":{"inode":1455085,"device":2049},"timestamp":"2017-09-27T17:45:47.828072524+03:00","ttl":-1}]
```

As you see, the registry file contains information about MySQL logs location, disc on which they are stored and the **inode** number of the file.

Once we have specified all required Filebeat settings mentioned above and restarted the Filebeat, it should begin sending information to our Elasticsearch cluster. To check whether Filebeat actually created new indices in our Elasticsearch, we can use Kibana. 

### **Kibana** 

![Image1]()

To find our MySQL logs in Elasticsearch, we first need to create an index pattern in Kibana management tab. The pattern for Filebeat logs is **filebeat-***. After saving the pattern, Kibana will show the list of your MySQL logs on the dashboard:

![Image2 displaying log entries in Kibana dashboard]()

As you can see, Filebeat transforms MySQL logs into *objects* that hold specific properties of logs such as timestamps, source file, log message, id and some others. This makes it easy to search and filter your MySQL logs using Elasticsearch queries. This approach also enables you to easily create visualizations in Kibana or fuel logs into AI/ML models and algorithms for pattern recognition and anomaly detection. To leverage this functionality, you can create custom fields that will track additional information from MySQL logs as per your requirements.

In the above example, we were sending MySQL logs directly to Elasticsearch, however, you can also send them to Logstash server. In Logstash, you can filter them, analyze, and eventually pass to Elasticsearch for storage. With Filebeat, this can be implemented with the following configuration.

```yaml
filebeat.prospectors:
- input_type: log
  paths:
    - /var/log/mysql/*.log
  document_type: syslog
  registry: /var/lib/filebeat/registry
output.logstash:
  hosts: ["mylogstashurl.example.com:5044"]
```

### Conclusion

As this tutorial demonstrates, Filebeat is an excellent log shipping solution for your MySQL database and Elasticsearch cluster. It's extremely lightweight compared to its predecessors as it comes to efficiently sending log events.  Filebeat supports compression and is easily configurable via a single **yaml** file. With Filebeat you can easily manage log files, keep track of log registry, create custom fields to enable granular filtering and discovery in your logs, and instantly power log data with Kibana visualization functionality. 

