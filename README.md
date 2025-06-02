# SQL to HDFS Data Pipeline

A gRPC-based data pipeline for ingesting, transforming, and analyzing loan data. It connects a MySQL database to an HDFS cluster using a custom Python server that supports SQL querying, Parquet conversion, distributed storage, and basic analytics — all containerized with Docker.

## Components

- MySQL Server: Hosts `loans` and `loan_types` tables
- gRPC Server: Handles SQL → HDFS flow and analytics
- HDFS Cluster: 1 NameNode and 3 DataNodes
- Client Interface: Triggers operations via gRPC
- Docker Compose: Orchestrates the stack

## Features

### DbToHdfs
- Connects to SQL
- Joins and filters loan records (`loan_amount` between 30,000 and 800,000)
- Converts result to Parquet using PyArrow
- Uploads to HDFS at `/hdma-wi-2021.parquet` with 2× replication

### BlockLocations
- Lists number of HDFS blocks stored per DataNode
- Uses WebHDFS `GETFILEBLOCKLOCATIONS`

### CalcAvgLoan
- Calculates average loan amount by county code
- Caches results as `partitions/<county>.parquet`
- Recreates data from master file if cache is missing
- Returns `"source"` field: `create`, `reuse`, or `recreate`

## HDFS File Layout
/hdma-wi-2021.parquet
/partitions/
├── 55001.parquet
├── 55003.parquet
└── ...

## Setup

### 1. Set up environment
```bash
export PROJECT=p4
```

### 2. Build Docker Images
```bash
docker build . -f Dockerfile.hdfs -t p4-hdfs
docker build . -f Dockerfile.namenode -t p4-nn
docker build . -f Dockerfile.datanode -t p4-dn
docker build . -f Dockerfile.mysql -t p4-mysql
docker build . -f Dockerfile.server -t p4-server
```

### 3. Start Services
```bash
docker compose up -d
```

## Usage
```bash
docker exec p4-server-1 python3 /client.py DbToHdfs
docker exec p4-server-1 python3 /client.py BlockLocations -f /hdma-wi-2021.parquet
docker exec p4-server-1 python3 /client.py CalcAvgLoan -c <county_code>
```

## Techstack
- Python 3
- gRPC
- MySQL
- HDFS with WebHDFS
- PyArrow
- Docker and Docker Compose
