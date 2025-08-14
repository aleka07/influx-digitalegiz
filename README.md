# InfluxDB + Telegraf Setup for DigitalEgiz

This repository contains a Docker Compose setup for running InfluxDB and Telegraf for the DigitalEgiz IoT project.

## Components

- **InfluxDB 2.7.6**: Time-series database for storing IoT sensor data
- **Telegraf 1.30.1**: Data collection agent for gathering metrics and sending them to InfluxDB

## Quick Start

### Prerequisites
- Docker
- Docker Compose
- Git

### Installation

1. Clone this repository:
```bash
git clone https://github.com/aleka07/influx-digitalegiz.git
cd influx-digitalegiz
```

2. **Create the data directory** (IMPORTANT - do this first!):
```bash
mkdir -p data/influxdb
sudo chown -R $USER:$USER data/
```

3. Create the external Docker network:
```bash
docker network create digital-egiz
```

4. Start the services:
```bash
docker-compose up -d
```

4. Check if services are running:
```bash
docker-compose ps
```

## Configuration

### InfluxDB Configuration
- **URL**: http://localhost:8086
- **Username**: admin
- **Password**: Aa31415926
- **Organization**: dtkaznu
- **Bucket**: iot-data

### Telegraf Configuration
The Telegraf configuration is stored in `telegraf.conf`. Modify this file to configure your data inputs and outputs.

## Accessing Data from Another Device

To access your InfluxDB data from another device on the same network:

### 1. Find Your Server's IP Address
On the host machine, run:
```bash
ip addr show | grep inet
```
Look for your local network IP (usually starts with 192.168.x.x or 10.x.x.x)

### 2. Configure Network Access

#### Option A: Bind to All Interfaces (Less Secure)
Edit `docker-compose.yml` and change the ports section for InfluxDB:
```yaml
ports:
  - "0.0.0.0:8086:8086"
```

#### Option B: Bind to Specific Interface (More Secure)
Replace `YOUR_SERVER_IP` with your actual server IP:
```yaml
ports:
  - "YOUR_SERVER_IP:8086:8086"
```

### 3. Update Firewall Rules
Allow access to port 8086:
```bash
# Ubuntu/Debian
sudo ufw allow 8086

# CentOS/RHEL
sudo firewall-cmd --permanent --add-port=8086/tcp
sudo firewall-cmd --reload
```

### 4. Access from Remote Device
Use the server's IP address instead of localhost:
- **InfluxDB URL**: `http://YOUR_SERVER_IP:8086`
- **Username**: admin
- **Password**: Aa31415926

### 5. Restart Services
After configuration changes:
```bash
docker-compose down
docker-compose up -d
```

## Data Persistence

**IMPORTANT**: The data directory must be created before starting the containers!

Data is persisted in the `./data/influxdb` directory on the host machine. This directory is excluded from Git to prevent committing sensitive data.

### Setting up the data directory:
```bash
# Create the directory structure
mkdir -p data/influxdb

# Set proper ownership (replace $USER with your username if needed)
sudo chown -R $USER:$USER data/

# Verify permissions
ls -la data/
```

**Note**: If you don't create this directory first, Docker will create it as root, which can cause permission issues.

## Useful Commands

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f influxdb
docker-compose logs -f telegraf
```

### Stop services
```bash
docker-compose down
```

### Restart services
```bash
docker-compose restart
```

### Access InfluxDB CLI
```bash
docker exec -it digitalegiz-influx influx
```

## Security Considerations

1. **Change default password**: The default password `Aa31415926` should be changed in production
2. **Network security**: Consider using VPN or SSH tunneling for remote access
3. **Firewall**: Only open necessary ports
4. **HTTPS**: Configure HTTPS for production use

## Troubleshooting

### Permission Issues with Data Directory
If you encounter permission issues:
```bash
# Stop containers first
docker-compose down

# Fix permissions
sudo chown -R $USER:$USER ./data

# Restart containers
docker-compose up -d
```

**Prevention**: Always create the data directory before starting containers:
```bash
mkdir -p data/influxdb
sudo chown -R $USER:$USER data/
```

### Network Issues
If containers can't communicate:
```bash
docker network ls
docker network inspect digital-egiz
```

### Port Conflicts
If port 8086 is already in use:
```bash
sudo netstat -tlnp | grep 8086
```

## Project Structure

```
influx-digitalegiz/
├── docker-compose.yml          # Main Docker Compose configuration
├── telegraf.conf              # Telegraf configuration
├── .gitignore                 # Git ignore rules
├── README.md                  # This file
└── data/                      # Data persistence (excluded from Git)
    └── influxdb/              # InfluxDB data directory (CREATE THIS FIRST!)
```

**Important**: The `data/influxdb/` directory must be created manually before running `docker-compose up` to avoid permission issues.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License.

## Support

For issues and questions, please create an issue in this repository.
