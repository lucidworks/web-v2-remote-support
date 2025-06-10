# Selenium Grid Hub with Lucidworks Connector

This repository contains a Docker Compose setup for running Selenium Grid with Chrome nodes and a Lucidworks connector for web scraping and content indexing.

> **Important Disclaimer**: The Selenium services (selenium-hub and chrome nodes) require x86 architecture to run properly. When running on ARM-based systems like Apple Silicon, these services WILL NOT WORK!!!

## System Requirements

- Docker Engine 19.03.0+
- Docker Compose 1.27.0+
- Operating Systems:
  - macOS (Intel/Apple Silicon) - Note: Platform compatibility warnings may appear on Apple Silicon but services should work
  - Linux (x86_64)
  - Windows with WSL2

## Configuration

### Docker Compose Configuration

The `docker-compose.yaml` file contains the following services:
- `selenium-hub`: The central hub for Selenium Grid
- `chrome-node-1` and `chrome-node-2`: Chrome browser nodes for running tests
- `lucidworks-connector`: Connector for web content indexing

### Required Files

Before starting, ensure you have the following files in the `bin` directory:
- `connector-plugin-standalone-5.9.7.jar` (or similar version)
- `lucidworks.connector.web-v2-2.0.1.zip` (or similar version)
- `conf/connector-config.yaml`
- `conf/logback.xml`

### Configuring the Lucidworks Connector

1. Edit `bin/conf/connector-config.yaml` to configure:
   - Kafka bridge settings
   - Proxy settings
   - Plugin path

Example configuration:
```yaml
kafka-bridge:
  target: your-connectors-backend.example.com:443
  plain-text: false
proxy:
  user: admin
  password: "yourpassword"
  url: https://your-fusion-instance.example.com/
plugin:
  path: /app/connector-plugin.zip
  type: fs
  suffix: remote
```

## Starting Docker Compose

### Starting in Background Mode

```bash
cd /path/to/selenium-grid-hub-docker-compose
docker-compose up -d
```

### Starting with Live Logs

To start all services and view logs in real-time:

```bash
cd /path/to/selenuim-grid-hub-docker-compose
docker-compose up
```

Press `Ctrl+C` to stop the services when viewing logs in real-time.

### Verifying Services

Once started, you can access:
- Selenium Grid Console: http://localhost:4444/ui
- The Lucidworks connector will be available on port 8764

### Viewing Live Logs

View logs for all services:
```bash
docker-compose logs -f
```

View logs for a specific service:
```bash
docker-compose logs -f selenium-hub
docker-compose logs -f chrome-node-1
docker-compose logs -f lucidworks-connector
```

Limit log output to recent entries:
```bash
docker-compose logs --tail=100
```

### Saving Logs to a File

```bash
docker-compose logs > docker-logs.txt
docker-compose logs lucidworks-connector > lucidworks-logs.txt
```

### Accessing Container-Specific Logs

To access logs directly from a container:
```bash
docker exec -it lucidworks-remote-connector cat /app/logs/connector.log
```
Note: This assumes logs are written to a file inside the container.

## Stopping Services

To stop all services:
```bash
docker-compose down
```

To stop and remove volumes (clearing all data):
```bash
docker-compose down -v
```

## Troubleshooting

### Common Issues

1. **Plugin Path Issues**: If the Lucidworks connector fails to start, check that the plugin path in `connector-config.yaml` is set to `/app/connector-plugin.zip` and the JAR file is correctly mounted in the container.

2. **Platform Compatibility Warnings**: On Apple Silicon Macs, you may see platform compatibility warnings for amd64 images. In most cases, the services will still function correctly through Docker's emulation layer.

3. **Connection Issues**: If the Lucidworks connector cannot connect to Kafka or Fusion, ensure the correct hostnames/URLs are set in `connector-config.yaml`.

### Checking Container Status

```bash
docker-compose ps
```

This will show the running status of all containers defined in your docker-compose file.