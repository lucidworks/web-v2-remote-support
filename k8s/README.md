# Selenium Grid Hub Setup

This repository contains configurations for running a Selenium Grid Hub with Chrome nodes in both Kubernetes environments.

## Overview

The Selenium Grid consists of:
- A Hub that routes WebDriver commands to appropriate nodes
- Chrome nodes that execute the browser automation tests
- Java application that connects to the Grid for automating web interactions

## Kubernetes Setup

The Kubernetes configurations include:

### 1. Selenium Hub Deployment
```yaml
# deployment.yaml
# Deploys the Selenium Hub component
```

### 2. Chrome Nodes Deployment
```yaml
# chrome-deployment.yaml
# Deploys 2 Chrome browser nodes that connect to the Hub
```

### 3. Service Configuration
```yaml
# service.yaml
# Exposes the Hub on ports 4444, 4442, and 4443
```

### Running in Kubernetes

To deploy to Kubernetes:

1. Apply the configurations:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f chrome-deployment.yaml
kubectl apply -f service.yaml
```

2. Verify the deployments:
```bash
kubectl get pods -n {{namespace}}
kubectl get services -n {{namespace}}
```
### Adjust the network policy
```bash
kubectl edit networkpolicy {{namespace}}-connector-plugin -n {{namepsace}}
```
add the following snippet
```
- ports:
  - port: 4444
    protocol: TCP
  - port: 4444
    protocol: UDP
```
## Notes

- The Chrome nodes use shared memory size of 2GB for better performance
- Each Chrome node supports up to 4 parallel sessions
- Session timeout is set to 60 seconds
- Grid timeout is set to 300 seconds
