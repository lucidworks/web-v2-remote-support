# noVNC Setup Guide for Selenium Grid

This guide explains how to enable noVNC (web-based VNC viewer) for your Selenium Grid setup, allowing you to visually monitor and debug browser sessions through a web interface.

## Table of Contents
- [What is noVNC?](#what-is-novnc)
- [Prerequisites](#prerequisites)
- [Step 1: Modify Docker Compose Configuration](#step-1-modify-docker-compose-configuration)
- [Step 2: Update Environment Variables](#step-2-update-environment-variables)
- [Step 3: Start Services with noVNC](#step-3-start-services-with-novnc)
- [Step 4: Access noVNC Interface](#step-4-access-novnc-interface)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)

## What is noVNC?

noVNC is a web-based VNC (Virtual Network Computing) client that allows you to view and interact with the desktop of your Chrome browser nodes directly from your web browser. This is particularly useful for:

- Debugging test execution in real-time
- Monitoring browser behavior during web scraping
- Visual verification of web page rendering
- Troubleshooting browser-specific issues

## Prerequisites

- Existing Selenium Grid setup (as described in the main [SETUP_GUIDE.md](SETUP_GUIDE.md))
- Docker and Docker Compose properly configured
- Web browser to access the noVNC interface

## Step 1: Modify Docker Compose Configuration

You need to update your `docker-compose.yaml` file to enable noVNC support. Below are the updated configurations for both version of the docker-compose jdk11 or jdk17.

### For "jdk11 or jdk17" Directory

Add the following environment variables and port mappings to your Chrome node services:

```yaml
# filepath: docker-compose/jdk11/docker-compose.yaml
version: '3'

services:
  selenium-hub:
    # ...existing hub configuration...

  chrome-node-1:
    image: selenium/node-chrome:4.20.0
    container_name: chrome-node-1
    shm_size: 2g
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_SESSION_TIMEOUT=60
    ports:
      - "5900:5900"  # VNC port
      - "7900:7900"  # noVNC web interface port
    networks:
      - selenium-grid
    restart: always

  chrome-node-2:
    image: selenium/node-chrome:4.20.0
    container_name: chrome-node-2
    shm_size: 2g
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_MAX_SESSIONS=4
      - SE_NODE_SESSION_TIMEOUT=60
    ports:
      - "7901:7900"  # noVNC web interface port (different from node-1)
    networks:
      - selenium-grid
    restart: always

  lucidworks-connector:
    # ...existing connector configuration...
```


## Step 2: Access noVNC Interface

Once the services are running, you can access the noVNC interfaces:

### Chrome Node 1
- **noVNC Web Interface**: http://localhost:7900

### Chrome Node 2
- **noVNC Web Interface**: http://localhost:7901

### Using the noVNC Interface

1. Open your web browser and navigate to http://localhost:7900 (for chrome-node-1)
2. You'll see the noVNC connection screen
3. Click "Connect" to establish the VNC connection
4. You should see the desktop of the Chrome node with a virtual display
5. Any browser sessions started on this node will be visible in this interface

### Example: Watching Browser Activity

To see noVNC in action:
1. Start the services with noVNC enabled
2. Open http://localhost:7900 in one browser tab
3. Open http://localhost:4444/ui (Selenium Grid Console) in another tab
4. Run your Lucidworks connector or any Selenium test
5. Watch the browser activity in real-time through the noVNC interface

## Troubleshooting

### Common Issues

1. **noVNC Interface Not Loading**:
   ```bash
   # Check if the container is running and ports are exposed
   docker-compose ps
   docker-compose logs chrome-node-1
   
   # Check if port 7900 is accessible
   curl -I http://localhost:7900
   ```

2. **VNC Connection Failed**:
   ```bash
   # Check container logs for VNC-related errors
   docker-compose logs chrome-node-1 | grep -i vnc
   
   # Ensure the container has enough resources
   docker stats chrome-node-1
   ```

3. **Performance Issues**:
   - Increase shared memory size: `shm_size: 4gb`
   - Reduce screen resolution for better performance
   - Limit concurrent sessions: `SE_NODE_MAX_SESSIONS=2`

### Advanced Troubleshooting

1. **Check VNC Process Inside Container**:
   ```bash
   docker exec -it chrome-node-1 ps aux | grep vnc
   ```

2. **Test VNC Connection Directly**:
   ```bash
   # Install VNC client and test direct connection
   # Ubuntu: sudo apt-get install vncviewer
   vncviewer localhost:5900
   ```

3. **Check Network Connectivity**:
   ```bash
   docker network ls
   docker network inspect selenium-grid
   ```

### Viewing Container Logs

```bash
# View noVNC-specific logs
docker-compose logs chrome-node-1 | grep -i vnc
docker-compose logs chrome-node-2 | grep -i vnc

# View all logs for a specific node
docker-compose logs -f chrome-node-1

# View startup logs
docker-compose logs chrome-node-1 | head -50
```


### Temporary Debug Mode

If you only need occasional debugging, create a separate docker-compose file:

```yaml
# docker-compose.debug.yaml
version: '3'
services:
  chrome-node-1:
    extends:
      file: docker-compose.yaml
      service: chrome-node-1
    environment:
      - SE_ENABLE_VNC=true
      - SE_NO_VNC=true
      - SE_VNC_NO_PASSWORD=1
    ports:
      - "7900:7900"
```

Then use it only when debugging:
```bash
docker-compose -f docker-compose.debug.yaml up -d
```

## Best Practices

1. **Resource Management**:
   - Monitor resource usage when noVNC is enabled
   - Consider disabling noVNC in production if not needed
   - Use appropriate screen resolutions for your use case

2. **Session Management**:
   - Limit concurrent sessions to prevent resource exhaustion
   - Monitor browser session lifecycle through noVNC

3. **Logging**:
   - Keep VNC logs for debugging purposes
   - Monitor connection attempts and failures

4. **Updates**:
   - Keep Selenium Docker images updated
   - Test noVNC functionality after image updates

---

**Note**: noVNC provides powerful debugging capabilities but consumes additional system resources. Use judiciously in production environments and always implement appropriate security measures.

For more information about Selenium Docker images and VNC configuration, visit the [official Selenium Docker documentation](https://github.com/SeleniumHQ/docker-selenium).
