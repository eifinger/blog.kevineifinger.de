---
layout: post
title: Updated my Homeassistant config with docker-compose and Screenshots
excerpt_separator: "<!--more-->"
categories: 
  - Archive
tags:
  - Homeassistant
---
I updated my homeassistant config with some screenshots and my extensive docker-compose.yaml file. Hope this helps some of you guys who want to use docker to run homeassistant.
<!--more-->

## docker-compose.yaml

I use docker-compose version 2.1 to make use of the *depends_on* feature.

```yaml
# Version > 3 does not work as it doesn't support 'depends_on'
version: '2.1'
services:
  homeassistant:
    container_name: homeassistant
    image: homeassistant/home-assistant:0.85.1
    volumes:
      - /home/admin/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro
    restart: always
    network_mode: host
    depends_on:
      influxdb:
        condition: service_healthy
      mysql-homeassistant:
        condition: service_healthy
      mosquitto:
        condition: service_started
  appdaemon:
    container_name: appdaemon
    restart: unless-stopped
    image: acockburn/appdaemon:3.0.2
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/admin/appdaemon:/conf
      - /home/admin/homeassistant/www:/config/www
    environment:
      - HA_URL="https://hidden.de"
      - TOKEN="secure"
      - DASH_URL="http://hidden:5050"
    ports:
      - "5050:5050"
      - "8124:8124"
# Does currently not work because I don't know how to properly handle multiline stacktraces
#    logging:
#      driver: splunk
#      options:
#        splunk-token: secure
#        splunk-url: https://splunk.kevineifinger.de:8088
#        splunk-insecureskipverify: "true"
#        splunk-verify-connection: "false"
    links:
      - facerec_service
    depends_on:
      - splunk
  facerec_service:
    container_name: facerec_service
    restart: unless-stopped
    image: eifinger/face_recognition:latest
    volumes:
     - /home/admin/facerec_service:/root/faces
    ports:
     - "9922:8080"
  influxdb:
    container_name: influxdb
    restart: unless-stopped
    image: influxdb:latest
    healthcheck:
      test: ["CMD", "curl", "-sI", "http://127.0.0.1:8086/ping"]
      interval: 30s
      timeout: 1s
      retries: 24
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/admin/influxdb:/var/lib/influxdb
    ports:
      - "8083:8083"
      - "8086:8086"
      - "8090:8090"
  grafana:
    container_name: grafana
    restart: unless-stopped
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD="secure"
      - GF_ANALYTICS_REPORTING_ENABLED=false
    ports:
      - "3000:3000"
    depends_on:
      - influxdb
  mysql-homeassistant:
    container_name: mysql-homeassistant
    restart: unless-stopped
    image: mysql:latest
    healthcheck:
        test: "/usr/bin/mysql --user=root --password=secure --execute \"SHOW DATABASES;\""
        interval: 2s
        timeout: 20s
        retries: 10
    environment:
      - MYSQL_ROOT_PASSWORD=secure
      - MYSQL_DATABASE=hass_db
      - MYSQL_USER=hassuser
      - MYSQL_PASSWORD=secure
    volumes:
      - /home/admin/mysql-homeassistant/mysql:/var/lib/mysql
    ports:
      - "3307:3306"
  mosquitto:
    container_name: mosquitto
    restart: unless-stopped
    image: eclipse-mosquitto
    volumes:
      - /home/admin/mosquitto/data:/mosquitto/data
      - /home/admin/mosquitto/log:/mosquitto/log
      - /home/admin/mosquitto/config:/mosquitto/config
    ports:
      - "1883:1883"
      - "9001:9001"
      - "8883:8883"
  splunk:
    container_name: splunk
    hostname: splunk
    restart: unless-stopped
    image: splunk/splunk:7.2
    volumes:
      - /home/admin/splunk/etc:/opt/splunk/etc
      - /home/admin/splunk/var:/opt/splunk/var
    environment:
      - SPLUNK_START_ARGS="--accept-license"
      - SPLUNK_ENABLE_LISTEN="9997"
      - SPLUNK_ADD="tcp 1514"
      - SPLUNK_PASSWORD="secure"
    ports:
      - "8001:8000"
      - "9997:9997"
      - "8088:8088"
      - "1514:1514"
volumes:
  grafana-storage:
```

## screenshots

Here are some teasers:

![default_view](https://raw.githubusercontent.com/eifinger/homeassistant-config/master/www/images/default_view.png)
![fuel_view](https://raw.githubusercontent.com/eifinger/homeassistant-config/master/www/images/fuel_view.png)
![sensor_view](https://raw.githubusercontent.com/eifinger/homeassistant-config/master/www/images/sensor_view.png)
![floorplan_view](https://raw.githubusercontent.com/eifinger/homeassistant-config/master/www/images/floorplan_view.png)

You can find my repo under [https://github.com/eifinger/appdaemon-config](https://github.com/eifinger/appdaemon-config)