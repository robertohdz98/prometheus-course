# Prometheus

This is a repository for several simple exercises about [**Prometheus**](https://prometheus.io/), an open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach. This is based on Technofor course for Prometheus Certified Associate (PCA) exam.


## 00. Simple Prometheus

In order to run a single Prometheus instance based on your own configuration, just set your Prometheus scrape configs, jobs and config in a `prometheus.yml` file and then mount it as a volume.

`docker run -d -p 9090:9090 -v ./prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus`


## 01. Prometheus environment

In order to run all the defined services, just run:

`docker-compose --env-file .env up`

An .env file is also provided to set the image tags (please check for compatibility).

Several services are then launched:

- **Prometheus**, reachable at http://localhost:9090, whose config file is set in *prometheus/* dir. Here, the services to be scraped and monitored are defined and are displayed in "Status > Targets":

![Scrapings](/figures/prom-scrapings.png)

- **Grafana**, reachable at http://localhost:3000;
- **MySQL** service;
- **MySQL Exporter**, reachable at http://localhost:9104, whose config file with credentials is set in *mysqld-exporter/* dir in order to avoid passing credentials as environment variables;
- **Alertmanager**, reachable at http://localhost:9093;
- **Node Exporter**, reachable at http://localhost:9100.

### 01.A Add Prometheus as data source in Grafana

Once our environment is set up, our goal is to connect Prometheus and Grafana in order to use this latter one to monitor our systems due to its more powerful visualization capabilities. To this end, access to our Grafana instance, change the default password and add our Prometheus instance as data source in "Home > Connections > Data Sources > Add data source > Prometheus" and type our Prometheus instance location `http://prometheus:9090`.

![DataSource](/figures/prom-grafana-datasource.png)

#### 01.B Our first dashboard in Grafana

Once our Prometheus instance is linked with Grafana, we can create dashboards to visualize Prometheus scraping metrics and monitor Prometheus scraped services, such as MySQL instance. To this end, our starting point is [an already created and publicly available dashboard for MySQL](https://grafana.com/grafana/dashboards/14031-mysql-dashboard/). In Grafana, click "Home > Dashboards > Import" > Copy Dashboard ID (e.g., 14031 in this case) and Load dashboard, where you can further personalize your panels.

![mysql-dashboard](/figures/grafana-mysql-dashboard.png)
