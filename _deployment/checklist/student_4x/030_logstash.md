---
title: Centralized Logging
permalink: "/deployment/checklist/student_4x/centralized_logging.html"
layout: "document"
categories: ["deployment", "checklist", "tds", "4x"]
---
# Install Elastic Search


The "ELK" stack, which stands for Elastic Search (ES), Logstash, and Kibana was leveraged for centralized logging. AWS has a packaged ES service, which includes Kibana as a plugin. Logstash was installed manually with the instructions below.  On AWS, ES was configured with IP-based security to only talk to our logstash hosts.  The Nginx reverse-proxy forwards kibana web requests over to the Kibana plugin.

An easy way to run a load balanced ES cluster of any size is by using AWS' Elasticsearch service. AWS has wrapped Elasticsearch with nice web configuration UI's that allow a user to easily create, re-configure, snapshot and monitor Elasticsearch. All you have to do is decide how many nodes of what class you want and how much disk space you need and their system does all the setup. You get a single endpoint to talk to the cluster.
You can resize and reconfigure the cluster while it is operating with no downtime. Their backend code will create a new cluster with your new config, migrate the data over, the decommission the old cluster without any downtime or loss of data.
Elasticsearch service setup

To set up the AWS ES service, navigate to the 'Amazon Elasticsearch Service dashboard' in your AWS account and click 'Create a new domain'. Then you fill in the form and click page and click Next until you're done! In our config, we're using IP-based security. To do this, select the 'allow access from specific IP' template during setup and put your logstash EC2 host IP addresses in the form (see the manual deployment section below for instructions on setting up logstash in EC2). We are currently using ES version 5.3, which is highly recommended as it's the first version that supports the curator tool, which we detail below.

Our load test ES cluster consists of 7 total nodes. 4 nodes are data nodes and 3 are dedicated master nodes. Having dedicated master nodes is highly recommended by AWS for performance, and 3 is the default number, and a good value to start with. We chose the m4.large.elasticsearch node type for all 7 nodes, but experimentation could be done to save cost or increase performance in your cluster. Perhaps use c4.large for the masters, as they need good compute and network speed but less storage performance than the data nodes.

Our current load test cluster gives 300GB to each data node, giving a 1.2TB total data size for the cluster. All of this is easy to change at runtime, and requires a 15-30 minute processing time, depending on the data sizes being migrated. Again, the cluster remains live and responsive throughout the migration process.

Any time you change the cluster configuration, it will show a status of 'processing' until the new cluster is set up. Once complete it will show 'active'.  At this point you can access the endpoint and kibana URL's that are given on the cluster details page in the dashboard.  To test the service from a lower level, you can log into whatever IP you gave ES access to above, and try:

`$ curl search-tds-dev-lrc7fjx6extulv62tfpgnwy5l4.us-west-2.es.amazonaws.com`

(use whatever your endpoint URL is - it listens on the standard HTTP port 80)
You should see some json like:
```
{
 "name" : "G717gG6",
 "cluster_name" : "269146879732:tds-dev",
 "cluster_uuid" : "VTdArnu9Rr-Si0A2HNS4Eg",
 "version" : {
   "number" : "5.1.1",
   "build_hash" : "5395e21",
   "build_date" : "2016-12-15T22:47:19.049Z",
   "build_snapshot" : false,
   "lucene_version" : "6.3.0"
 },
 "tagline" : "You Know, for Search"
} 
```

# Kibana setup with nginx

As the ES cluster will only listen to the provided IP, you'll need to use something like nginx to bounce requests to the logstash host over to the kibana plugin in the ES cluster. To do this:

`$ sudo apt install nginx`

`$ sudo systemctl enable nginx.service`

Then edit `/etc/nginx/sites-enabled/default` and add the following at the top (use your ES endpoint URL):
```
    upstream es {
        ip_hash;
        server search-sbtds-load2-yiozpjgoawn2denvnprhozceaq.us-west-2.es.amazonaws.com;
        keepalive 128;
    }
```
    
And place this inside the `server {}` block:
```
    location /_plugin/kibana/ {
        proxy_pass http://es/_plugin/kibana/;
        auth_basic "closed site";
        auth_basic_user_file kibana.htpasswd;
        proxy_http_version 1.1;
        proxy_set_header Authorization "";
        proxy_set_header Connection "";
    }
``` 
Then, to password protect your kibana instance, add a file to `/etc/nginx` called `kibana.htpasswd` with the following contents (season to taste - this file creates a user kibana with password newpass):
```
  # basic auth
  kibana:{PLAIN}newpass
```  
This sets up open, plain, cleartext password authentication over unencrypted http. For a production instance, it's recommended to use stronger security over SSL, perhaps with the users in an LDAP or other directory.  In our environment, we then added a DNS name (with AWS route 53) to point to our enabled IP address. After setting this up, you should be able to access Kibana at the following URL:

`http://your-dns-name.your-domain.org/_plugin/kibana/`

# Configuring logstash to talk to the AWS ES service

After setting up logstash as described below, you can edit your `tds-pipeline.conf` file to include the ES endpoint URL instead of the IP's used in our example. Our example has this elasticsearch {} stanza in the output section. You don't want a port number, as it runs on the default HTTPS port 443 (you do need to put the https:// on the front of the URL):
```
  # AWS managed elasticsearch
  elasticsearch {
      hosts => [
          "https://search-tds-dev-lrc7fjx6extulv62tfpgnwy5l4.us-west-2.es.amazonaws.com"
      ]
  }
```
# Logstash installation and configuration

Logstash can be installed using APT, as follows:

`$ sudo apt install logstash`

Running as a service

You may want to configure systemctl to start up the service at boot time. 
To configure logstash to start automatically when the system boots up, run the following commands:

`$ sudo /bin/systemctl daemon-reload`

`$ sudo /bin/systemctl enable logstash.service`


Logstash can be started and stopped as follows:

`$ sudo systemctl start logstash.service`

`$ sudo systemctl stop logstash.service`

# Logstash configuration

The following values must be defined in `/etc/logstash/logstash.yml`:

```
http.host: 172.31.1.237
path.config: /etc/logstash/conf.d
path.data: /var/lib/logstash
path.logs: /var/log/logstash
```

The file `/etc/logstash.conf.d/tds-pipeline.conf` contains specific rules for TDS. A sample `tds-pipeline.conf` is included below.

These TDS rules specify the following:
*	Log entries are received from logbeats on TCP port 5043.
*	Log entries are received in json format on TCP port 4560.
*	A log message matching the format "EVENT:<event_name> JSON:(<json_data>)" is an 'event' and will go to elastic search.
*	All log messages of level WARN or ERROR will go to elastic search.
*	All logs (including those that don't match the above) are collected, gzip compressed, and uploaded to S3 every 15 minutes, into the S3 bucket 'tds-logstash-dev'.

```
input {
    beats {
        port => 5043
    }
    tcp {
        port => 4560
        codec => json_lines
    }
}
filter {
    grok {
        match => { message => "EVENT:%{GREEDYDATA:event} JSON:\((?<JSONDATA>.*)\)" }
    }
    json {
        source => "JSONDATA"
    }
}
output {
    if [level] == "ERROR" or [level] == "WARN" or "_grokparsefailure" not in [tags] {
      elasticsearch { 
        hosts => [
            "172.31.15.103:9200",
            "172.31.0.214:9200",
            "172.31.43.9:9200",
            "172.31.30.99:9200"
        ]
      }
    }
    s3 {
	id => "tds_everything"
        bucket => "tds-logstash-dev"
        region => "us-west-2"
        aws_credentials_file => "/etc/logstash/aws_credentials_file"
        size_file => 5242880
        time_file => 15
        canned_acl => "private"
        tags => ["tds", "all", "logstash-dev-2"]
        storage_class => "STANDARD" # might want "STANDARD_IA" to save
        encoding => "gzip"
	codec => "json"
    }
    # stdout { codec => rubydebug }
}
```

The TDS rule file refers to another file: `/etc/logstash/aws_credentials_file`, which shall contain AWS credentials in this format: 
```
:access_key_id: "YOUR_ACCESS_KEY"
:secret_access_key: "YOUR_SECRET_ACCESS_KEY" 
```

Scaling Note from https://www.elastic.co/guide/en/logstash/5.0/deploying-and-scaling.html: Make sure that your Logstash configuration does not connect directly to Elasticsearch dedicated master nodes, which perform dedicated cluster management. Connect Logstash to client or data nodes to protect the stability of your Elasticsearch cluster.

# Long term Elastic Search data maintenance

It's important to periodically remove old information from elastic search so the cluster doesn't grow too large.  One tool that makes this easy is called 'curator', and it supported by Elastic. We have provided some simple curator actions that can help with this process.

Curator is documented at `https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html`, and can be installed as follows:

`$ sudo apt install python3-pip`
`$ sudo pip install elasticsearch-curator`

Configure curator by customizing the curator configuration file as described here: https://www.elastic.co/guide/en/elasticsearch/client/curator/current/configfile.html
Next, place the attached cleanup_action.yml in a convenient directory and execute the action as follows:

`$ curator cleanup_action.yml`

This command will delete all indices over 90 days old, and close all indices over 45 days old. These values can, of course, be tweaked in the action file. Closed indices have virtually no runtime cost to elastic search, but can easily be opened again should the need arise. Deleted indices, however, are completely removed and not recoverable.
It is recommended that this command is configured to run in a daily crontab from an elastic search master, kibana server, or other host with convenient access to your elastic search cluster.
Another way to expire data in elastic search is to set a TTL for each index. This is outside the scope of this document, but easily accomplished and is automatic once set. The curator method described here is preferred, as it takes effect regardless of the individual metadata settings inside each elastic search index, and can therefore be viewed as more reliable in bulk. The values in the cleanup action file can also be changed at will and made effective immediately by executing the curator. Once could set up the TTL to a long period as a failsafe against very old data clogging up the data store in case the curator crontab fails to run. Once data older than expected is noticed in elastic search, once could then investigate any potential curator issue.

[back to Deployment Checklists](index.html)