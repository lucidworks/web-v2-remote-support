# Web V2 Remote Connector Setup Guide

This document provides step-by-step instructions for setting up and running the Lucidworks Web V2 Remote Connector with Selenium Grid Hub for web content indexing.

## Table of Contents
- [Prerequisites](#prerequisites)
- [File Structure](#file-structure)
- [Step 1: Download Required Files](#step-1-download-required-files)
- [Step 2: Choose the Right Version](#step-2-choose-the-right-version)
- [Step 3: Configure the Connector](#step-3-configure-the-connector)
- [Step 4: Start the Docker Compose Environment](#step-4-start-the-docker-compose-environment)
- [Step 5: Verify Services](#step-5-verify-services)
- [Troubleshooting](#troubleshooting)
- [Stopping Services](#stopping-services)

## Prerequisites

Ensure you have the following installed on your system:
- Docker Engine 19.03.0+
- Docker Compose 1.27.0+
- Compatible Operating System:
  - Linux (x86_64)
  - Windows with WSL2

> **Important**: The Selenium services (selenium-hub and chrome nodes) require x86 architecture. They will not work properly on ARM-based systems like Apple Silicon Macs.

## File Structure

The repository is organized into two main directories based on the connector plugin version:
- `docker-compose/jdk17` - For connector plugin standalone version 5.9.11 and newer
- `docker-compose/jdk11` - For connector plugin standalone versions older than 5.9.10

Each directory contains:
- `docker-compose.yaml` - Docker Compose configuration file
- `bin/` - Directory to store required JAR and ZIP files
  - `conf/` - Configuration files
    - `connector-config.yaml` - Connector configuration
    - `logback.xml` - Logging configuration

## Step 1: Download Required Files

1. Visit the [Lucidworks Plugins website](https://plugins.lucidworks.com/) to download the required files.

2. From the website, download the following files:

   a. **Web V2 Connector Plugin**:
      - Find the latest version of `lucidworks.connector.web-v2` from the "Current plugin version" section
      - Current latest version: `lucidworks.connector.web-v2-2.1.0.zip` (Released: Apr 24, 2025)

   b. **Connector Plugin Standalone Jar**:
      - Choose the appropriate version based on your requirements
      - Latest version: `connector-plugin-standalone-5.9.12.jar` (Released: Apr 16, 2025)
      - Other available versions: 5.9.11, 5.9.10, 5.9.9, 5.9.8, 5.9.7, etc.

## Step 2: Choose the Right Version

Based on the connector plugin standalone version you downloaded, choose the appropriate directory:

1. If you downloaded version 5.9.10 or newer:
   ```bash
   cd "5.9.10 and greater"
   ```

2. If you downloaded a version older than 5.9.10:
   ```bash
   cd "lower than 5.9.10"
   ```

> **Note**: The main difference between these setups is the Java version used in the Docker image. Newer connector versions (5.9.10+) use Java 17, while older versions use Java 11.

## Step 3: Configure the Connector

1. Edit the connector configuration file:
   ```bash
   vim bin/conf/connector-config.yaml
   ```
   
2. Update the following settings:

   ```yaml
   kafka-bridge:
     target: your-connectors-backend.example.com:443
     # Uncomment proxy-server section if needed
   
   proxy:
     user: admin
     password: "yourpassword"
     url: https://your-fusion-instance.example.com/
   
   plugin:
     path: /app/connector-plugin.zip
     type:
      suffix: remote-
   ```

   - `kafka-bridge.target`: Your Fusion Connectors backend service
   - `proxy.user`: Fusion admin username
   - `proxy.password`: Fusion admin password
   - `proxy.url`: URL to your Fusion instance

3. Save the configuration file.

## Step 4: Start the Docker Compose Environment

### Option 1: Start in Background Mode

```bash
docker-compose up -d
```

This starts all services in detached mode (background).

### Option 2: Start with Live Logs

```bash
docker-compose up
```

This starts all services and displays logs in the terminal. Press `Ctrl+C` to stop.

## Step 5: Verify Services

1. **Check Selenium Grid Console**:
   - Open a web browser and navigate to: http://localhost:4444/ui
   - Verify that the Selenium Hub is running and Chrome nodes are connected

2. **Verify Lucidworks Connector**:
   - The connector will be available on port 8764
   - Check logs to ensure proper startup:
     ```bash
     docker-compose logs lucidworks-connector
     ```

3. **Check Container Status**:
   ```bash
   docker-compose ps
   ```
   All containers should show as "Up" status.

## Troubleshooting

### Common Issues

1. **Missing Required Files**:
   - Ensure you have placed the connector-plugin-standalone JAR and lucidworks.connector.web-v2 ZIP in the bin directory
   - Verify file permissions are correct

2. **Configuration Errors**:
   - Check `connector-config.yaml` for correct settings
   - Verify URL formats and credentials

3. **Connection Issues**:
   - If the connector cannot connect to Kafka or Fusion, ensure the correct hostnames/URLs are set in `connector-config.yaml`
   - Check network connectivity to your Fusion instance

4. **Plugin Path Issues**:
   - Ensure the plugin path in `connector-config.yaml` is set to `/app/connector-plugin.zip`
   - Verify the JAR file is correctly mounted in the container

5. **Platform Compatibility Warnings**:
   - On Apple Silicon Macs, you may see platform compatibility warnings for amd64 images
   - In most cases, the Lucidworks connector will still work through Docker's emulation layer

### Viewing Logs

```bash
# View all logs
docker-compose logs -f

# View logs for a specific service
docker-compose logs -f lucidworks-connector

# View recent log entries
docker-compose logs --tail=100

# Save logs to a file
docker-compose logs > docker-logs.txt
docker-compose logs lucidworks-connector > lucidworks-logs.txt

# Access container-specific logs
docker exec -it lucidworks-remote-connector cat /app/logs/connector.log
```

## Stopping Services

To stop all services:
```bash
docker-compose down
```

To stop and remove volumes (clearing all data):
```bash
docker-compose down -v
```

---

This setup guide is based on configurations from June 2025. For updated versions of connectors and plugins, please check the [Lucidworks Plugins website](https://plugins.lucidworks.com/).
