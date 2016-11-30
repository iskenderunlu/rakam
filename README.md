[![Build Status](https://travis-ci.org/rakam-io/rakam.svg?branch=master)](https://travis-ci.org/rakam-io/rakam) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="url" alt="alt text" width="whatever" height="whatever"> [<img alt="Deploy" src="http://www.herokucdn.com.rsz.io/deploy/button.png" height="21">](https://dashboard.heroku.com/new?button-url=https%3A%2F%2Fgithub.com%2Frakam-io%2Frakam&template=https%3A%2F%2Fgithub.com%2Frakam-io%2Frakam) [![Install on DigitalOcean](http://installer.71m.us/button.svg)](http://installer.71m.us/install?url=https://github.com/rakam-io/rakam)
Rakam
=======
[![Join user group at https://groups.google.com/forum/#!forum/rakam](https://img.shields.io/badge/google-groups-green.svg?style=flat)](https://groups.google.com/forum/#!forum/rakam)
[![Join the chat at https://gitter.im/rakam-io/rakam](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/buremba/rakam?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Rakam is an analytics platform that allows you to create your analytics services.

Features / Goals
------------
Rakam is a modular analytics platform that gives you a set of features to create your own analytics service.

Typical workflow of using Rakam:
* Collect data from multiple sources with **[trackers, client libraries, webhooks, tasks etc.](//rakam.io/doc/buremba/rakam-wiki/master/Collecting-Events)**
* Enrich and sanitize your event data with **[event mappers](//rakam.io/doc/buremba/rakam-wiki/master/Event-Mappers)**
* Process data in real-time using **[real-time modules](//rakam.io/doc/buremba/rakam-wiki/master/Modules#realtimeanalyticsmodulesubapidocapirakamiorealtimesub)** (pre-aggregate data via stream processing using SQL!),
* Store data in a data warehouse to analyze it later. (Postgresql, HDFS, S3 etc.)
* Analyze your event data with your SQL queries and integrated rich analytics APIs (**[funnel, retention, real-time reports](//rakam.io/doc/buremba/rakam-wiki/master/Analyze-Data)**, **[event streams](//rakam.io/doc/Modules#eventstreammodulesubapidocgetrakamcomapitagsstreamsub)**)
* Analyze your users with **[integrated CRM tool](//rakam.io/doc/buremba/rakam-wiki/master/Modules#customeranalyticsmodulesubapidocapirakamiousersub)**
* Create custom dashboards and real-time reports using **[Rakam BI](https://rakam.io/)**
* **[Develop your own modules](//rakam.io/doc/buremba/rakam-wiki/master/Developing-Modules)** for Rakam to customize it for your needs.

All these features come with a single box, you just need to specify which modules you want to use using a configuration file (config.properties) and Rakam will do the rest for you.
We also provide cloud deployment tools for scaling your Rakam cluster easily.

Deployment
----------

If your event data-set can fit in a single server, we recommend using Postgresql backend. Rakam will collect all your events in row-oriented format in a Postgresql node. All the features provided by Rakam are supported in Postgresql deployment type.

However Rakam is designed to be highly scalable in order to provide a solution for high work-loads. You can configure Rakam to send events to a distributed commit-log such as Apache Kafka or Amazon Kinesis in serialized Apache Avro format and process data in PrestoDB workers and store them in a distributed filesystem in a columnar format.

### Heroku

You can deploy Rakam to Heroku using Heroku button, it uses Heroku Postgresql add-on for your app and uses Postgresql deployment type.

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://dashboard.heroku.com/new?button-url=https%3A%2F%2Fgithub.com%2Frakam-io%2Frakam&template=https%3A%2F%2Fgithub.com%2Frakam-io%2Frakam)

### Digitalocean

Digitalocean installer uses Docker under the hood. It will install Docker on your droplet and run docker image of Postgresql and link it with your Rakam container. The API the image exposes is `9999`, you may visit your Rakam API address from `YOUR_DROPLET_IP:9999`. The default lock key is `mylockkey`.

[![Install on DigitalOcean](http://installer.71m.us/button.svg)](http://installer.71m.us/install?url=https://github.com/rakam-io/rakam)

### Docker

Run the following command to start a Postgresql server in docker container and Rakam API in your local environment.

    docker run -d --name rakam-db -e POSTGRES_PASSWORD=dummy -e POSTGRES_USER=rakam postgres:9.6.1 && docker run --link rakam-db --name rakam -p 9999:9999 -e RAKAM_CONFIG_LOCK-KEY=mylockkey buremba/rakam

After docker container is started, visit [http://127.0.0.1:9999](http://127.0.0.1:9999) and follow the instructions. You can also register your local Rakam API to Rakam BI at
[http://app.rakam.io/cluster/register](http://app.rakam.io/cluster/register?apiUrl=http:%2F%2F127.0.0.1:9999&lockKey=mylockKey)
or directly use Rakam API. You may also consult to [API documentation](https://api.rakam.io) for details of the API.

We also provide docker-compose definition for Postgresql backend. Create a `docker-compose.yml` with from this definition and run the command  `docker-compose run rakam-api -f docker-compose.yml`.

    version: '2'
    services:
      rakam-db:
        image: postgres:9.6.1
        environment:
          - POSTGRES_PASSWORD=dummy
          - POSTGRES_USER=rakam
      rakam-api:
        image: buremba/rakam
        ports:
          - "9999:9999"
        depends_on:
          - rakam-db

You can set config variables for Rakam instance using environment variables. All properties in config.properties file can be set via environment variable `RAKAM_CONFIG_property_name_dots_replaced_by_underscore`.
For example, if you want to set `store.adapter=postgresql` you need to set environment variable `RAKAM_CONFIG_STORE_ADAPTER=postgresql`.

Dockerfile will generate `config.properties` file from environment variables in docker container that start with `RAKAM_CONFIG` prefix.

In order to set environment variables for container, you may use `-e` flag for for `docker run` but we advice you to set all environment variables in a file and use  `--env-file` flag when starting your container.

Then you can share same file among the Rakam containers. If Dockerfile can't find any environment variable starts with `RAKAM_CONFIG`, it tries to connect Postgresql instance created with docker-compose.

### AWS (Cloudformation)

Postgresql deployment type: (Best for < 1B events):
[![Deploy to AWS](https://d0.awsstatic.com/product-marketing/Elastic%20Beanstalk/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=ParseBackend&templateURL=https://s3.amazonaws.com/rakam-prod-static/cloudformation/rakam-postgresql.template)

If the data volume is big and you need support, please [contact us](https://rakam.io/contact) for Cloudformation template of Rakam for Presto deployment type.

Cloudformation templates create a Opsworks stack in your AWS account for Rakam. You can easily monitor, scale and manage your Rakam cluster with these Cloudformation templates.

Cloudformation is the recommended way to deploy Rakam in production because AWS automatically handles most of the complexity like fail over and load-balancing.

### Custom

Download latest version from [Bintray](https://dl.bintray.com/buremba/maven/org/rakam/rakam), ([VERSION]/rakam-[VERSION]-.bundle.tar.gz) extract package, modify `etc/config.properties` file and run `bin/launcher start`.
The launcher script can take the following arguments: `start|restart|stop|status|run`. 
`bin/launcher run` will start Rakam in foreground.

### Managed

We're also working for managed Rakam cluster, we will deploy Rakam to our AWS accounts and manage it for you so that you don't need to worry about scaling, managing and software updates. We will do it for you.
Please shoot us an email to `emre@rakam.io` if you want to test our managed Rakam service.

Web application
------------
This repository contains Rakam API server that allows you to interact with Rakam using a REST interface. If you already have a frontend and developed a custom analytics service based on Rakam, it's all you need.

However, we also developed Rakam Web Application that allows you to analyze your user and event data-set but performing SQL queries, visualising your data in various charts, creating (real-time) dashboards and custom reports. You can turn Rakam into a analytics web service similar to [Mixpanel](https://mixpanel.com), [Kissmetrics](https://kissmetrics.com) and [Localytics](https://localytics.com) using the web application. Otherwise, Rakam server is similar to [Keen.io](https://keen.io) with SQL as query language and some extra features.

Another nice property of Rakam web application is being BI `(Business Intelligence)` tool. If you can disable collect APIs and connect Rakam to your SQL database with JDBC adapter and use Rakam application to query your data in your database. Rakam Web Application has various charting formats, supports parameterized SQL queries, custom pages that allows you to design pages with internal components.

Contribution
------------
Currently I'm actively working on Rakam. If you want to contribute the project or suggest an idea feel free to fork it or create a ticket for your suggestions. I promise to respond you ASAP.
The purpose of Rakam is being generic data analysis tool which can be a solution for many use cases. Rakam still needs too much work and will be evolved based on people's needs so your thoughts are important.

Acknowledgment
--------------
[![YourKit](https://www.yourkit.com/images/yklogo.png)](https://www.yourkit.com/java/profiler/index.jsp)

We use YourKit Java Profiler in order to monitor the JVM instances for identifing the bugs and potential bottlenecks. Kudos to YourKit for supporting Rakam with your full-featured Java Profile!
