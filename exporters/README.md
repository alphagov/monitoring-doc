# Prometheus exporters via Docker compose

## Introduction

There is a number of exporters that are provided by the Prometheus team in order to ensure you can collect application specific metrics. You can find the available exporters at the following location.

There is official exporters & non-official exporters available to you. We have created this virtual docker compose environment that you can use in order to make sure you are able to try & test explore exporters within Grafana. We have also provided the ability to explore metrics via Grafana to help better understanding how to craft custom queries.

## Requirements

 * Docker compose 
 * Docker 
 * Available ports:
    * Prometheus: 9090
    * Alertmanger: 9093
    * Grafana: 3000

## Services & Exporters

 * prometheus
 * Redis
 * Memcached
 * Postgresql
 * Elasticsearch
 * MysqlD
 * MongoDB
 * RabbitMQ
 * Grafana

Each one of these services is accompanied by the appropriate exporter that will automatically start scraping the services & import the relevant data into Prometheus. Grafana will then be available for you to explore those metrics. You can access Grafana via localhost port 3000. There is also the Prometheus dashboard that you can access via localhost:9090.

## Bringing up the core environment

The core environment consists of a few components which enable you to fully test the functionality of Prometheus. These components are the ones that have ports directly exposed & allow you to perform the required testing. You can also bring up additional components which will allow you to test additional functionality for that particular service & its exporters.

In order to start the environment all that is required is to run:    ``` docker-compose -f exporter-docker-configs.yaml up -d``` .
This assumes that all the requirements have been satisfied. You may need sudo permissions depending on how you hVW installed docker. You can go directly to either localhost:3000 for Grafana or localhost:9090 to access the Prometheus dashboard.

## Adding additional components

When bringing up an enviroment if you would like to add additional components you would use the following command: ``` docker-compose -f exporter-docker-configs.yaml -f <<component-name>>-exporter.yaml up -d``` 

You would use an extra ```-f``` in order to add an extra component to the stack that you are deploying. The compoents name that are avalible in this stack are the following: 

 * Redis
 * Memcached
 * Postgresql
 * Elasticsearch
 * MysqlD
 * MongoDB
 * RabbitMQ

## Setting up the environment

The Grafana instance is not properly configured. We need to set it up so that it can scrape metrics from Prometheus. We do this by adding a data source. You need to do the following:
1. Click on the Grafana logo on the top left-hand side. 
2. Click on the data source 
3. Select Add data source on the top right-hand side
4. Pick a name for your data source:
    * Select type Prometheus
    * In HTTP setting enter the URL: http://prometheus:9090
    * Leave 'proxy' configured as the access type
    * Click on Add to the datatype
5. Once this step is completed you can then add your desired dashboard

## Tearing down the environment

In order to destroy the environment, you need to run the following command: ``` docker-compose -f exporter-docker-configs.yaml down ```  