# Docker Compose Setup
This repository contains a `docker-compose.yml` file that sets up multiple services for local development. Below is an explanation of each service and how to use it.

## Services

### Ubuntu

- **Image**: `ubuntu:latest`
- **Container Name**: `ubuntu-local-server`
- **Command**: `/usr/sbin/sshd -D`
- **Ports**: `2222:22`
- **Environment Variables**:
    - `USER`: root
    - `PASSWORD`: root
- **Volumes**:
    - `ubuntu_data:/data`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs an Ubuntu container with SSH enabled. You can SSH into the container using:

```sh
ssh root@localhost -p 2222
```
Password:
```sh
root
```

### MariaDB

- **Image**: `mariadb:latest`
- **Container Name**: `mariadb`
- **Ports**: `3306:3306`
- **Environment Variables**:
    - `MYSQL_ROOT_PASSWORD`: `${DB_PASSWORD:-root}`
    - `MYSQL_DATABASE`: `${DB_DATABASE:-demo}`
    - `MYSQL_USER`: `${DB_USERNAME:-root}`
    - `MYSQL_PASSWORD`: `${DB_PASSWORD:-root}`
    - `MYSQL_ALLOW_EMPTY_PASSWORD`: `1`
- **Volumes**:
    - `mariadb_data:/var/lib/mysql`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs a MariaDB database. You can connect to it using:

```sh
mysql -h localhost -P 3306 -u root -p
```

### phpMyAdmin

- **Image**: `phpmyadmin/phpmyadmin:latest`
- **Container Name**: `phpmyadmin`
- **Depends On**: `mariadb`
- **Ports**: `8080:80`
- **Environment Variables**:
    - `PMA_ARBITRARY`: `1`
    - `PMA_HOST`: `mariadb`
    - `PMA_PORT`: `3306`
    - `PMA_PMADB`: `${DB_DATABASE:-demo}`
    - `PMA_USER`: `${DB_USERNAME:-root}`
    - `PMA_PASSWORD`: `${DB_PASSWORD:-root}`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs phpMyAdmin for managing MariaDB. Access it at:

```sh
http://localhost:8080
```

### PostgreSQL

- **Image**: `postgres:latest`
- **Container Name**: `postgres`
- **Ports**: `5432:5432`
- **Environment Variables**:
    - `POSTGRES_DB`: `${POSTGRES_DB:-demo}`
    - `POSTGRES_USER`: `${DB_USERNAME:-root}`
    - `POSTGRES_PASSWORD`: `${DB_PASSWORD:-root}`
- **Volumes**:
    - `postgres_data:/var/lib/postgresql/data`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs a PostgreSQL database. You can connect to it using:

```sh
psql -h localhost -p 5432 -U root -d demo
```

### pgAdmin

- **Image**: `dpage/pgadmin4:latest`
- **Container Name**: `pgadmin`
- **Depends On**: `postgres`
- **Ports**: `5050:80`
- **Environment Variables**:
    - `PGADMIN_DEFAULT_EMAIL`: `${PGADMIN_DEFAULT_EMAIL:-admin@admin.com}`
    - `PGADMIN_DEFAULT_PASSWORD`: `${DB_PASSWORD:-admin}`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs pgAdmin for managing PostgreSQL. Access it at:
```sh
http://localhost:5050
```

### Redis

- **Image**: `redis:latest`
- **Container Name**: `redis`
- **Ports**: `6379:6379`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs a Redis server. You can connect to it using:

```sh
redis-cli -h localhost -p 6379
```

### Redis Commander

- **Image**: `rediscommander/redis-commander:latest`
- **Container Name**: `redis-commander`
- **Depends On**: `redis`
- **Ports**: `8081:8081`
- **Environment Variables**:
    - `REDIS_HOSTS`: `local-redis:redis`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs Redis Commander for managing Redis. Access it at:
```sh
http://localhost:8081
```
### MinIO

- **Image**: `minio/minio`
- **Container Name**: `minio`
- **Ports**:
    - `9000:9000`
    - `9001:9001`
- **Environment Variables**:
    - `MINIO_ROOT_USER`: `${MINIO_ROOT_USER:-admin}`
    - `MINIO_ROOT_PASSWORD`: `${MINIO_ROOT_PASSWORD:-password123}`
- **Volumes**:
    - `minio_data:/data`
- **Command**: `server /data --console-address ":9001"`
- **Healthcheck**:
    - `test`: `["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]`
    - `interval`: `30s`
    - `timeout`: `10s`
    - `retries`: `5`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs MinIO for object storage. Access the MinIO console at:
```sh
http://localhost:9001
```
### SonarQube

- **Image**: `sonarqube:latest`
- **Container Name**: `sonarqube`
- **Networks**: `localnet`
- **Restart Policy**: always

This service runs SonarQube for code quality and security analysis. Access it at:
```sh
http://localhost:9002
```
### Credentials

- **Username**: `admin`
- **Password**: `admin`


---

## Networks

- **localnet**: A custom network for all services to communicate.

## Volumes

- `ubuntu_data`
- `mariadb_data`
- `postgres_data`
- `minio_data`

## Usage

To start all the services, run:
```sh
docker-compose up -d
```

To stop all the services, run:
```sh
docker-compose down
```

To view logs for a specific service, run:
```sh
docker-compose logs <service_name>
```
Replace `<service_name>` with the name of the service (e.g., `mariadb`, `redis`).

## Environment Variables

You can customize the environment variables by creating a `.env` file in the root directory. The following variables are used:

- `DB_PASSWORD`
- `DB_DATABASE`
- `DB_USERNAME`
- `POSTGRES_DB`
- `PGADMIN_DEFAULT_EMAIL`
- `MINIO_ROOT_USER`
- `MINIO_ROOT_PASSWORD`

Example `.env` file:

```env
DB_PORT=3307
DB_DATABASE=sqlmydatabase
DB_USERNAME=root
DB_PASSWORD=password

POSTGRES_DB=pgmydatabase
PGADMIN_DEFAULT_EMAIL=email@mail.com

MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=password123
```
