---
layout: post
title: "Playing with Prometheus"
date: 2020-07-26
description:
image: /assets/images/prometheus.png
author: Fabio Bertagna
tags:
  - Prometheus
---
Nach der Theorie zu Prometheus und Grafana in den [/sys-labs]({{ site.baseurl }}/2020/07/24/sys-labs) habe ich diese beiden Technologien in einem ganz einfachen Docker-Setup zusammen gespannt, um diese mal in der Praxis auszuprobieren und damit rum zu spielen.

In einem Container soll eine Prometheus Server Instanz laufen, welche metrics von einem weiteren Container holen soll. Dieser Container ist ein Linux System mit dem Node Exporeter drauf welcher Prometheus daten zur verfügung stellt. Grafana soll in einem dritten Container laufen, wo dann der Prometheus Container als Data Source registriert werden soll, und in einem Dashboard die Daten visualisiert werden sollen.

Im `docker-compose.yml` file sieht das Ganze wie folgt aus.

```yml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./config:/etc/prometheus/
      - ./data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/home/prometheus'
    expose:
      - 9090
    ports:
      - 9090:9090
    links:
      - target:target


  target:
    container_name: target
    image: prom/node-exporter:latest
    expose:
      - 9100
    ports:
      - 9100:9100

  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    links:
      - prometheus:prometheus
    restart: unless-stopped
    volumes:
      - ./data/grafana:/var/lib/grafana
    user: "root"
    ports:
      - 3000:3000

```

Mit diesem Setup hatte ich die Möglichkeit verschiedenes auszuprobieren und lernen, wie z.B. das registrieren einer Data Source bei Grafana, das Erstellen oder Importieren eines Dashboards auch in Grafana oder auch Queries über den Prometheus Server auf die Targets abzusetzen.

![Grafana Dashboard]({{ site.baseurl }}/assets/images/grafana-dashboard.png)

Bei diesem kleinen Experiment war nicht nur spannend zu sehen wie Prometheus und Grafana zusammen funktionieren, sondern auch das Konfigurieren der Docker-Container. Dabei war es wichtig die Container über die benötigten Ports füreinander erreichbar machen, damit jeder Container seine benötigten Informationen beim jeweiligen Container abholen kann.
