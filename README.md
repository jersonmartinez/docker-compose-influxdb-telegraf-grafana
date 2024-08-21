# Monitoring with Docker Compose using InfluxDB, Telegraf and Grafana

This repository contains a Docker Compose setup for creating containers for InfluxDB, Telegraf, and Grafana, providing a comprehensive monitoring solution.

## Infrastructure Model

![Infrastructure Model](https://github.com/user-attachments/assets/cbac355d-2c26-469b-a953-650b083a7eda)

## Quick Start

### Clone the Repository

```bash
git clone https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana.git
cd docker-compose-influxdb-telegraf-grafana
```

### Deploy the Stack

```bash
docker-compose up -d
```

![Deployment Output](https://github.com/user-attachments/assets/0a9f8a91-66f6-4aec-a71c-16f5d6051b59)

## Detailed Setup Guide

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana.git
   cd docker-compose-influxdb-telegraf-grafana
   ```

2. **Start the Services**:
   ```bash
   docker-compose up -d
   ```

3. **Verify the Services**:
   ```bash
   docker-compose ps
   ```

4. **Access Grafana**:
   - Open your browser and navigate to `http://localhost:3000`
   - Default credentials: username and password are both `admin`

## Using the Repository

### Configuring Telegraf

- Telegraf configuration files are located in the `telegraf` directory.
- After modifying the configuration files, restart the Telegraf container:
  ```bash
  docker-compose restart telegraf
  ```

### Viewing Metrics in Grafana

1. Open Grafana and add InfluxDB as a data source.
2. Create dashboards and panels to visualize the metrics collected by Telegraf.

## Telegraf Configuration for Different Environments

We provide detailed guides for configuring Telegraf in various environments:

- [Telegraf on Ubuntu](docs/install_telegraf_on_ubuntu.md)
- [Telegraf on Windows Subsystem for Linux (WSL) - Debian](docs/install_telegraf_on_wsl_linux.md)
- [Telegraf on Windows](docs/install_telegraf_on_windows.md)
- [Telegraf in Docker](docs/explaining_telegraf.md#monitorizar-contenedores-docker)

For a comprehensive explanation of Telegraf configuration across different platforms, refer to our [Telegraf Configuration Guide](docs/explaining_telegraf.md).

## Service Descriptions

1. **InfluxDB**: A time-series database designed for high write and query loads, used to store metrics collected by Telegraf.

2. **Telegraf**: An agent for collecting, processing, aggregating, and writing metrics. It gathers data from the host and Docker containers, sending it to InfluxDB.

3. **Grafana**: A web-based interface for visualizing metrics stored in InfluxDB, allowing creation of dashboards and panels for monitoring.

4. **Nginx**: A web server providing static content serving and request proxying capabilities.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.