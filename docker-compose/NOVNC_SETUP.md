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

You need to update your `docker-compose.yaml` file to enable noVNC support. Below are the updated configurations for both version directories:

### For "5.9.10 and greater" Directory

Add the following environment variables and port mappings to your Chrome node services:

```yaml
# filepath: 5.9.10 and greater/docker-compose.yaml
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
      - SE_NODE_MAX_SESSIONS=4
      - SE_NODE_SESSION_TIMEOUT=60
      # Enable noVNC
      - SE_ENABLE_VNC=true
      - SE_NO_VNC=true
      - SE_VNC_NO_PASSWORD=1
      - SE_SCREEN_WIDTH=1920
      - SE_SCREEN_HEIGHT=1080
      - SE_SCREEN_DEPTH=24
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
      # Enable noVNC
      - SE_ENABLE_VNC=true
      - SE_NO_VNC=true
      - SE_VNC_NO_PASSWORD=1
      - SE_SCREEN_WIDTH=1920
      - SE_SCREEN_HEIGHT=1080
      - SE_SCREEN_DEPTH=24
    ports:
      - "5901:5900"  # VNC port (different from node-1)
      - "7901:7900"  # noVNC web interface port (different from node-1)
    networks:
      - selenium-grid
    restart: always

  lucidworks-connector:
    # ...existing connector configuration...
```

### For "lower than 5.9.10" Directory

The configuration is identical, just place it in the appropriate directory:

```yaml
# filepath: lower than 5.9.10/docker-compose.yaml
# Same configuration as above
```

## Step 2: Update Environment Variables

Here's an explanation of the key environment variables added:

| Variable | Description | Default Value | Recommended Value |
|----------|-------------|---------------|-------------------|
| `SE_ENABLE_VNC` | Enables VNC server | `false` | `true` |
| `SE_NO_VNC` | Enables noVNC web interface | `false` | `true` |
| `SE_VNC_NO_PASSWORD` | Disables VNC password authentication | `0` | `1` (for dev/testing) |
| `SE_SCREEN_WIDTH` | Screen width in pixels | `1360` | `1920` |
| `SE_SCREEN_HEIGHT` | Screen height in pixels | `1020` | `1080` |
| `SE_SCREEN_DEPTH` | Color depth | `24` | `24` |

### Optional: Enable VNC Password Protection

If you want to add password protection to your VNC sessions (recommended for production):

```yaml
environment:
  - SE_ENABLE_VNC=true
  - SE_NO_VNC=true
  - SE_VNC_NO_PASSWORD=0  # Enable password
  - SE_VNC_PASSWORD=your_secure_password  # Set your password
  - SE_SCREEN_WIDTH=1920
  - SE_SCREEN_HEIGHT=1080
  - SE_SCREEN_DEPTH=24
```

## Step 3: Start Services with noVNC

1. **Navigate to the appropriate directory**:
   ```bash
   # For connector-plugin-standalone-5.9.10 or newer
   cd "5.9.10 and greater"
   
   # OR for older versions
   cd "lower than 5.9.10"
   ```

2. **Stop existing services** (if running):
   ```bash
   docker-compose down
   ```

3. **Start services with the new configuration**:
   ```bash
   # Start in background mode
   docker-compose up -d
   
   # OR start with live logs
   docker-compose up
   ```

4. **Verify services are running**:
   ```bash
   docker-compose ps
   ```

   You should see all services in "Up" status, including the new port mappings for 5900/5901 and 7900/7901.

## Step 4: Access noVNC Interface

Once the services are running, you can access the noVNC interfaces:

### Chrome Node 1
- **noVNC Web Interface**: http://localhost:7900
- **Direct VNC**: `localhost:5900` (using VNC client like RealVNC, TightVNC)

### Chrome Node 2
- **noVNC Web Interface**: http://localhost:7901
- **Direct VNC**: `localhost:5901` (using VNC client)

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

3. **Port Conflicts**:
   ```bash
   # Check if ports are already in use (macOS/Linux)
   lsof -i :7900
   lsof -i :7901
   lsof -i :5900
   lsof -i :5901
   
   # On Windows
   netstat -an | findstr :7900
   netstat -an | findstr :7901
   ```

4. **Screen Resolution Issues**:
   - Adjust `SE_SCREEN_WIDTH` and `SE_SCREEN_HEIGHT` environment variables
   - Common resolutions: 1920x1080, 1600x1200, 1366x768
   - Restart containers after making changes:
     ```bash
     docker-compose down
     docker-compose up -d
     ```

5. **Performance Issues**:
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
   # macOS: brew install --cask vnc-viewer
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

## Security Considerations

### Development vs Production

For **development environments**:
- Using `SE_VNC_NO_PASSWORD=1` is acceptable for convenience
- Binding to all interfaces (`0.0.0.0`) is usually fine

For **production environments**, implement these security measures:

1. **Enable VNC Password Protection**:
   ```yaml
   environment:
     - SE_VNC_NO_PASSWORD=0
     - SE_VNC_PASSWORD=your_secure_password_here
   ```

2. **Restrict Network Access**:
   ```yaml
   ports:
     - "127.0.0.1:7900:7900"  # Only allow localhost access
     - "127.0.0.1:5900:5900"  # Only allow localhost access
   ```

3. **Use Docker Networks**:
   ```yaml
   # Remove port mappings and access via reverse proxy
   # ports:
   #   - "7900:7900"  # Comment out for production
   ```

4. **Implement Reverse Proxy with Authentication**:
   ```nginx
   # nginx configuration example
   server {
       listen 80;
       server_name vnc.yourdomain.com;
       
       auth_basic "VNC Access";
       auth_basic_user_file /etc/nginx/.htpasswd;
       
       location / {
           proxy_pass http://localhost:7900;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
       }
   }
   ```

### Firewall Configuration

Ensure these ports are properly configured:
- `5900, 5901` - VNC direct access (restrict to localhost or VPN)
- `7900, 7901` - noVNC web interface (restrict to localhost or VPN)
- `4444` - Selenium Grid Console (can be more open for team access)

### Alternative: Temporary Debug Mode

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
docker-compose -f docker-compose.yaml -f docker-compose.debug.yaml up -d
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
