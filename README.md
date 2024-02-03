# Prometheus

This is a repository for several simple exercises about [**Prometheus**](https://prometheus.io/), an open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach. It is based on **Technofor course for Prometheus Certified Associate (PCA) exam**.

[Link to PCA Certification](https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/578a4927-7b3e-42cd-80f1-4fa015563c94-roberto-hernandez-ruiz-c92fde45-2216-48b5-a58c-9e1f37fdf30c-certificate.pdf)

This Prometheus repo consists of:
- **Environment based on Prometheus** service using `docker-compose` to create several services or targets to be scraped by Prometheus
- Configuring several **Prometheus exporters**
- **Leveraging Grafana's powerful visualization capabilities**, linking it with Prometheus to create dashboards for MySQL, Redis, Node Exporter... monitoring (using **PromQL**)
- Managing alerts from **Alertmanager** (Prometheus) and Grafana
- **Prometheus federation**, which allows a Prometheus server to scrape selected time series from another Prometheus server.
- **Prometheus & Grafana stack** manifest files for deployment in **Kubernetes** (`/k8s-deployment`).

<br/>

# Table of Contents
[00. Simple Prometheus](#00-simple-prometheus)

[01. Prometheus environment](#01-prometheus-environment)
- [01.A Add Prometheus as data source in Grafana](#01.A-add-prometheus-as-data-source-in-grafana)
- [01.B Our first dashboards in Grafana](#01.B-our-first-dashboards-in-grafana)
- [01.C Alerts in Grafana](#01.C-playing-with-alerts-in-grafana)
- [01.D Alerts in Prometheus and Alertmanager](#01.D-playing-with-alerts-in-prometheus-and-alertmanager)

[02. Prometheus federation](#02-prometheus-federation)

<br/>
<br/>

# 00. Simple Prometheus

In order to run a single Prometheus instance based on your own configuration, just set your Prometheus scrape configs, jobs and config in a `prometheus.yml` file and then mount it as a volume.

`docker run -d -p 9090:9090 -v ./prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus`

<br/>
<br/>

# 01. Prometheus environment

In order to run all the defined services, just run:

`docker-compose -f docker-compose.yml --env-file .env up`

An .env file is also provided to set the image tags (please check for compatibility).

Several services are then launched:

- **Prometheus**, reachable at http://localhost:9090, whose config file is set in *prometheus/* dir. Here, the services to be scraped and monitored are defined and are displayed in "Status > Targets":

![Scrapings](/figures/prom-scrapings.png)

- **Grafana**, reachable at http://localhost:3000;
- **MySQL** service;
- **MySQL Exporter**, reachable at http://localhost:9104, whose config file with credentials is set in *mysqld-exporter/* dir in order to avoid passing credentials as environment variables;
- **Alertmanager**, reachable at http://localhost:9093;
- **Node Exporter**, reachable at http://localhost:9100.
- **Redis** and its associated **Redis Exporter** (reachable at http://localhost:9121)
- Other additional services can be included, such as Blackbox Exporter (reachable at http://localhost:9115).

<br/>

## 01.A Add Prometheus as data source in Grafana

Once our environment is set up, our goal is to connect Prometheus and Grafana in order to use this latter one to monitor our systems due to its more powerful visualization capabilities. To this end, access to our Grafana instance, change the default password and add our Prometheus instance as data source in "Home > Connections > Data Sources > Add data source > Prometheus" and type our Prometheus instance location `http://prometheus:9090`.

![DataSource](/figures/prom-grafana-datasource.png)

<br/>

### 01.B Our first dashboards in Grafana

Once our Prometheus instance is linked with Grafana, we can create dashboards to visualize Prometheus scraping metrics and monitor Prometheus scraped services, such as MySQL instance. To this end, our starting point is [an already created and publicly available **dashboard for MySQL**](https://grafana.com/grafana/dashboards/14031-mysql-dashboard/). In Grafana, click "Home > Dashboards > Import" > Copy Dashboard ID (e.g., 14031 in this case) and Load dashboard, where you can further personalize your panels.

![mysql-dashboard](/figures/grafana-mysql-dashboard.png)

Besides, we could have several dashboards to monitor different services. Here, **a dashboard for Redis based on Redis Exporter** is created (ID 673):

![redis-dashboard](https://github.com/robertohdz98/prometheus-certification-course/assets/68640342/bdb9d0dc-b2ab-4219-b580-6b77111d499c)

Another Grafana **dashboard based on Node Exporter** (ID 1860):

![node-exporter-dashboard](https://github.com/robertohdz98/prometheus-certification-course/assets/68640342/b8e6c3fb-6fc5-46b9-973b-5de0f3861ba2)

But, additionally and as an alternative to this latest dashboard, in this mini-project for the course a custom, more simple dashboard (whose JSON is located in `grafana/node-exporter-dashboard.json` in this repo) has been created for some metrics scraped **using Prometheus and Node Exporter** through PromQL queries.

For example, the "Memory Usage Percentage" panel in the right shows the percentage of memory usage of the host (PromQL query: `100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100`). In this specific panel, some custom thresholds as filled regions have been designed to easily visualize when a lot of tasks are being performed and when the memory is filling up (could add alerts here!):

![my-node-exporter-dashboard](/figures/my-node-exporter-dashboard.png)

<br/>

### 01.C Playing with alerts in Grafana

Besides, Grafana and Prometheus allow to set personalized alerts (MS Teams, mail, etc.) according to several queries or events. In this lab, as an example, a simple alert is created in Grafana for those cases when MySQL reaches a custom limit of client connections. This limit has been arbitrarily set to 2 clients (i.e., alert raises if a third client connects to MySQL). The alert is configured in a specific panel of the dashboard using PromQL (see screenshot below), and then it is needed to set an Alert Contact Point (e.g., MS Teams, e-mail, etc.) in "Alerting > Contact points" menu.

<img width="760" alt="alert-mysql-clients" src="https://github.com/robertohdz98/prometheus-examples/assets/68640342/b1602334-e662-4f53-b463-c4c58aaec771">

It is also possible to see that an alert has fired in the panel due to 3 connections:

<img width="901" alt="Screenshot 2023-11-01 at 21 26 52" src="https://github.com/robertohdz98/prometheus-examples/assets/68640342/e8841077-4614-4e78-af7d-57e6bb486be4">

<br/>

### 01.D Playing with alerts in Prometheus and Alertmanager

Similarly, Prometheus can also set alerts based on events or changes in the state of its scraped services with the help of **Alertmanager**. There are two steps here:

- Firstly, **defining the rules based on queries that will trigger the alerts**. This is possible by configuring the alert rules in a .yml file (here `prometheus/alerts.yml`) and then referencing those rules
from the Prometheus configuration file using `rule_files` flag. The configured alerts can be visualized in "Alerts" from the Prometheus UI.

![alerts-prometheus](https://github.com/robertohdz98/prometheus-certification-course/assets/68640342/17dcf687-03d8-4d61-9cdb-73746e717999)

- Secondly, **managing those alerts**. Prometheus itself does not handle alerts, it only sets those alerts and trusts on **Alertmanager** for these "warning" tasks (Grafana is able to do both).

<br/>
<br/>

# 02. Prometheus federation

Federation allows a Prometheus server to scrape selected time series from another Prometheus server.
To recreate this environment, just launch:

`docker-compose -f docker-compose.fed.yml --env-file .env up`

This `docker-compose.fed.yml` file configures to another Prometheus environment where several services are created:

- 2 isolated Prometheus instances `prometheus1` and  `prometheus2`, which are reachable at http://localhost:9091 and http://localhost:9092 respectively;

![federate-1](/figures/federate-1.png)
![federate-2](/figures/federate-2.png)

- 2 MySQL servers and 2 MySQL exporters, one for each isolated Prometheus (see them as targets in screenshots above);
- A "master" Prometheus instance which gathers and collects metrics from those federated Prometheus and their respective scraped services, reachable at http://localhost:9099 (see screenshot of its targets below);

![federate-master](/figures/federate-master.png)

- Node Exporter as an additional service that could be included.

As a result, **all metrics could be gathered and monitored from a single Prometheus instance**:

![federate-all](/figures/federate-all.png)
