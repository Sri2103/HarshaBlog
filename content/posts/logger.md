+++
title = 'Logger'
date = 2024-12-17T13:19:52-05:00
author = "Harsha"
draft = false
tags = ["docker","js","javascript","monitoring","metrics","elasticsearch","kibana","prometheus","grafana"]
+++

<!-- markdown-link-check-disable -->
## Building a Logging and Monitoring Architecture with Kubernetes, Logstash, Elasticsearch, and Kibana

## Introduction

Logs are a critical part of any system, providing insight into application behavior, debugging information, and security monitoring. This blog post documents the design and implementation of a logging and monitoring system using a microservices architecture in Kubernetes. It integrates Logstash, Elasticsearch, Kibana, and Prometheus, with a Node.js web service (Primary Service) and a Sidecar container for filtering logs.

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Design](#architecture-design)
3. [Setting Up the Environment](#setting-up-the-environment)
4. [Implementation Details](#implementation-details)
   - [Primary Service (Node.js Web App)](#primary-service-nodejs-web-app)
   - [Sidecar Container (Log Filter)](#sidecar-container-log-filter)
5. [Centralized Logging with Logstash](#centralized-logging-with-logstash)
6. [Viewing Logs in Kibana](#viewing-logs-in-kibana)
7. [Monitoring with Prometheus and Grafana](#monitoring-with-prometheus-and-grafana)
8. [Testing the Setup](#testing-the-setup)
9. [Conclusion](#conclusion)

## System Overview

The logging system ensures:

- Application logs are filtered for critical warnings (`WARN`) and errors (`ERROR`) before being sent to Logstash.
- Filtered logs are indexed in Elasticsearch for storage and retrieval.
- Visualizations are created using Kibana.
- Metrics are collected using Prometheus and displayed on Grafana dashboards.

This system uses a Kubernetes setup, but the components are tested locally using Docker Compose.

### **Architecture Design**

The architecture employs the following components:

- **Primary Service**: A Node.js web application generating logs.
- **Sidecar Container**: Filters `WARN` and `ERROR` logs from shared log files and sends them to Logstash.
- **Logstash**: Processes logs from the Sidecar and forwards them to Elasticsearch.
- **Elasticsearch**: Stores logs for indexing and retrieval.
- **Kibana**: Visualizes logs stored in Elasticsearch.
- **Prometheus**: Monitors application metrics, including HTTP request counts and latencies.
- **Grafana**: Displays Prometheus metrics via dashboards.

<!-- markdown-link-check-disable -->
[![](https://mermaid.ink/img/pako:eNpFkEFqwzAQRa8iZp1cwItCErsuNIVSQxaVs5hKE0vUssxICoSQu1dR3HZWmvf-YvSvoLwmqGBgnI3Yf_STyLOR72wd8kV0xGer6CjW6yexlZ1BJi0OfkyOjo_wtrid7KwmhbzQXaG13PshRAxmwXXBjWxGDNGqQMjq1zXFPctX-4UTLnBTYJsP8o6ioRQW0RbxIlvG0z0OK3DEDq3O37neMz3kvKMeqvzUyN899NMt5zBF310mBVXkRCtgnwYD1QnHkLc0a4xUW8yduD864_Tp_f9O2kbPb4_2Som3H6ecaQQ?type=png)](https://mermaid.live/edit#pako:eNpFkEFqwzAQRa8iZp1cwItCErsuNIVSQxaVs5hKE0vUssxICoSQu1dR3HZWmvf-YvSvoLwmqGBgnI3Yf_STyLOR72wd8kV0xGer6CjW6yexlZ1BJi0OfkyOjo_wtrid7KwmhbzQXaG13PshRAxmwXXBjWxGDNGqQMjq1zXFPctX-4UTLnBTYJsP8o6ioRQW0RbxIlvG0z0OK3DEDq3O37neMz3kvKMeqvzUyN899NMt5zBF310mBVXkRCtgnwYD1QnHkLc0a4xUW8yduD864_Tp_f9O2kbPb4_2Som3H6ecaQQ)

## Setting Up the Environment

### **Prerequisites**

- Docker and Docker Compose
- Nodejs (for the Primary Service and Sidecar)

## Implementation Details

### Primary Service (Node.js Web App)

The Primary Service is a Node.js web application that generates logs. It uses the `winston` library for logging, with a custom `warn` and `error` transport.

- Logs ``INFO``, ``WARN``, and ``ERROR`` messages to a shared file.
- Exposes Prometheus metrics for HTTP requests.

Code for app.js:

```javascript

const express = require("express");
const fs = require("fs");
const path = require("path");
const { trackDuration, metricsHandler } = require("./promClient");

const app = express();
const logFilePath = path.join(__dirname, "logs", "app.log");

// Middleware to track request duration for Prometheus
app.use(trackDuration);

// Log request details to a file
app.use((req, res, next) => {
  const logMessage = `${new Date().toISOString()} - ${req.method} ${req.url}\n`;
  fs.appendFileSync(logFilePath, logMessage);
  next();
});

// Simple home route
app.get("/", (req, res) => {
  res.send("Hello, world! This is the primary service.");
});

// Prometheus metrics route
app.get("/metrics", metricsHandler);

// Simulate error and warning logs
app.get("/warn", (req, res) => {
  const logMessage = `${new Date().toISOString()} - WARN - Something might be wrong!\n`;
  fs.appendFileSync(logFilePath, logMessage);
  res.send("Warning logged.");
});

app.get("/error", (req, res) => {
  const logMessage = `${new Date().toISOString()} - ERROR - Something went wrong!\n`;
  fs.appendFileSync(logFilePath, logMessage);
  res.status(500).send("Error logged.");
});

// Start the server
const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Primary service listening on port ${port}`);
});

```

code for promClient.js:

```javascript
const promClient = require("prom-client");

// register the client
const register = new promClient.Registry();

const httpRequestDurationMicroseconds = new promClient.Histogram({
  name: "http_request_duration_ms",
  help: "Duration of HTTP requests in ms",
  labelNames: ["method", "route", "code"],
  buckets: [0.1, 5, 15, 50, 100, 200, 300, 400, 500], // buckets for response time from 0.1ms to 500ms
});

register.registerMetric(httpRequestDurationMicroseconds);

// collect default metrics
promClient.collectDefaultMetrics({ register });

// Request duration tracker function
const trackDuration = (req, res, next) => {
  const end = httpRequestDurationMicroseconds.startTimer();
  res.on("finish", () => {
    end({
      method: req.method,
      route: req.route.path,
      code: res.statusCode,
    });
    console.log("Request duration tracked");
  });

  next();
};

const metricsHandler = async(req, res) => {
  res.set("Content-Type", register.contentType);
  data = await register.metrics();
  await res.end(data);
};

module.exports = {
  trackDuration,
  metricsHandler,
};
```

### Sidecar Container (Log Filter)

The Sidecar Container filters `WARN` and `ERROR` logs from the shared log file and sends them to Logstash.

Code for sidecar.js:

```javascript
const fs = require('fs');
const path = require('path');

// Configuration
const configPath = path.join(__dirname, 'sidecar-config', 'config.json');
const config = JSON.parse(fs.readFileSync(configPath, 'utf-8'));

// Log file paths
const inputLogFile = path.join('/app/logs', 'app.log'); // Shared volume log file
const outputLogFile = path.join('/app/logs', 'filtered.log'); // Filtered logs

// Watch the input log file
fs.watchFile(inputLogFile, { interval: 1000 }, () => {
    const logs = fs.readFileSync(inputLogFile, 'utf-8').split('\n');
    const filteredLogs = logs.filter((log) => {
        // Apply log filtering based on level
        return config.logLevels.some((level) => log.includes(level));
    });

    // Write filtered logs to the output file
    fs.writeFileSync(outputLogFile, filteredLogs.join('\n'), { flag: 'w' });
    console.log(`Filtered logs written to ${outputLogFile}`);
});

console.log('Sidecar container started, watching for logs...');

```

### Centralized Logging with Logstash

Logstash processes logs from the Sidecar and forwards them to Elasticsearch.

### Viewing Logs in Kibana

- Kibana visualizes logs stored in Elasticsearch.
- To view logs, navigate to the Kibana dashboard at `http://localhost:5601`.

### Monitoring with Prometheus and Grafana

- Prometheus collects metrics from the Primary Service.
- Grafana displays metrics from Prometheus.

### Testing the Setup

- Run the Docker Compose setup.
- Access the Primary Service at `http://localhost:3000`.
- View logs in Kibana at `http://localhost:5601`.
- Monitor metrics in Grafana at `http://localhost:3001`.

## Conclusion

This project demonstrates how to implement centralized logging and monitoring in microservices architecture. The integration of Logstash, Elasticsearch, and Kibana ensures efficient log management, while Prometheus and Grafana provide real-time application monitoring.

Feel free to clone the GitHub repository and customize the setup for your use case!
<!-- add github link here -->
[Github link](<https://github.com/Sri2103/logsProvider>)

---
