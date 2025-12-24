# db-collector

Lightweight database metrics collector for DBPilot. Runs on your database server and pushes metrics to the DBPilot platform.

**Version:** 0.0.2

## Supported Databases

- PostgreSQL 12+
- MySQL 5.7+ / MariaDB 10.3+

## Installation

### Prerequisites

1. Get your API key from the DBPilot dashboard
2. Note your namespace and datastore names

### Install via Script

```bash
curl -fsSL https://raw.githubusercontent.com/dbpilot/db-collector/main/scripts/install.sh | sudo bash -s -- \
  --api-key="YOUR_API_KEY" \
  --namespace="production" \
  --datastore="main-cluster" \
  --node-id="$(hostname)" \
  --api-endpoint="https://api.dbpilot.io"
```

### Install Options

| Option | Required | Description |
|--------|----------|-------------|
| `--api-key` | Yes | Your DBPilot API key |
| `--namespace` | Yes | Environment name (e.g., production, staging) |
| `--datastore` | Yes | Database cluster name |
| `--node-id` | Yes | Unique name for this server (defaults to hostname) |
| `--api-endpoint` | No | DBPilot API URL |
| `--ca-cert` | No | Path to CA certificate for self-signed certs |
| `--tls-skip-verify` | No | Skip TLS verification (development only) |

### Post-Installation

1. **Create database monitoring user:**

```sql
-- PostgreSQL
CREATE USER dbpilot_collector WITH PASSWORD 'your_secure_password';
GRANT pg_monitor TO dbpilot_collector;

-- MySQL
CREATE USER 'dbpilot_collector'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT SELECT, PROCESS, REPLICATION CLIENT ON *.* TO 'dbpilot_collector'@'localhost';
```

2. **Save the password:**

```bash
echo "your_secure_password" | sudo tee /opt/db-collector/config/.db_password > /dev/null
sudo chmod 600 /opt/db-collector/config/.db_password
```

3. **Start the collector:**

```bash
sudo systemctl enable db-collector
sudo systemctl start db-collector
```

## Usage

```bash
# Check status
sudo systemctl status db-collector

# View logs
sudo journalctl -u db-collector -f

# Restart
sudo systemctl restart db-collector
```

## Configuration

Config file: `/opt/db-collector/config/config.yaml`

```yaml
node:
  id: "pg-prod-01"
  namespace: "production"
  datastore: "main-cluster"

database:
  type: "postgresql"  # or "mysql"
  host: "localhost"
  port: 5432
  user: "dbpilot_collector"
```

## Troubleshooting

**Collector won't start:**
```bash
sudo journalctl -u db-collector -n 50
```

**Check database connection:**
```bash
# PostgreSQL
sudo -u db-collector psql -U dbpilot_collector -d postgres -c "SELECT 1;"

# MySQL
mysql -u dbpilot_collector -p -e "SELECT 1;"
```

**Check API connectivity:**
```bash
curl -I https://api.dbpilot.io/health
```

## Uninstall

```bash
sudo systemctl stop db-collector
sudo systemctl disable db-collector
sudo rm -rf /opt/db-collector
sudo rm /etc/systemd/system/db-collector.service
sudo systemctl daemon-reload
```

## Resources

- CPU: <1%
- Memory: ~50MB
- Network: ~10 KB/s (compressed)

## License

Copyright 2025 DBPilot. All rights reserved.
