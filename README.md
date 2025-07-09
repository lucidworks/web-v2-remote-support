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
- `lucidworks-connector`: The service where the connector web-v2 will be running

See the [SETUP_GUIDE.md](SETUP_GUIDE.md) for detailed instructions on how to set up and configure the Lucidworks connector with Selenium Grid.

Optionally, See the [NOVNC_SETUP.md](NOVNC_SETUP.md) for instructions on how to set up noVNC support for remote access to the Selenium Grid nodes.