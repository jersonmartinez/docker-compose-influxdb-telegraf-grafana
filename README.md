# Monitoring with Docker Compose using InfluxDB, Telegraf and Grafana
This is a docker compose to create containers about InfluxDB, Telegraf and Grafana.

### Infrastructure model

![image](https://github.com/user-attachments/assets/cbac355d-2c26-469b-a953-650b083a7eda)

#### Clone the repository:
```bash copy
git clone https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana.git
cd docker-compose-influxdb-telegraf-grafana
```

#### Apply:

```bash
docker-compose up -d
```

![image](https://github.com/user-attachments/assets/83421ba9-6f86-4539-877d-92a2ff9ac4ff)

### Setting up the repository

1. **Clone the repository**:
   ```bash
   git clone https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana.git
   cd docker-compose-influxdb-telegraf-grafana
   ```

2. **Start the services**:
   ```bash
   docker-compose up -d
   ```

3. **Verify the services**:
   - Check if the containers are running:
     ```bash
     docker-compose ps
     ```

4. **Access Grafana**:
   - Open your browser and go to `http://localhost:3000`
   - Default username and password are both `admin`

### Using the repository

1. **Accessing InfluxDB**:
   - Open your browser and go to `http://localhost:8086`
   - Default username and password are both `admin`

2. **Configuring Telegraf**:
   - Telegraf configuration files are located in the `telegraf` directory.
   - Modify the configuration files as needed and restart the Telegraf container:
     ```bash
     docker-compose restart telegraf
     ```

3. **Viewing metrics in Grafana**:
   - Open Grafana and add InfluxDB as a data source.
   - Create dashboards and panels to visualize the metrics collected by Telegraf.

### Purpose of each service

1. **InfluxDB**:
   - InfluxDB is a time-series database designed to handle high write and query loads. It is used to store the metrics collected by Telegraf.

2. **Telegraf**:
   - Telegraf is an agent for collecting, processing, aggregating, and writing metrics. It collects metrics from the host and Docker containers and sends them to InfluxDB.

3. **Grafana**:
   - Grafana is a web-based interface for visualizing metrics stored in InfluxDB. It allows you to create dashboards and panels to monitor the metrics collected by Telegraf.

4. **Nginx**:
   - Nginx is a web server that provides a web server to serve static content and proxy requests.
