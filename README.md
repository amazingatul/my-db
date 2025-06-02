# Complete Linux Database Setup Guide (From Scratch)

## Prerequisites
- An active AWS account
- Basic knowledge of Linux commands
- SSH client (like Terminal or PuTTY)

## 🖥️ Step 1: Launch an EC2 Instance
1. Go to AWS EC2 Console
2. Click on **Launch Instance**
3. Configure:
   - **Name**: db-server
   - **Amazon Machine Image (AMI)**: Ubuntu Server 22.04 LTS (or Amazon Linux)
   - **Instance type**: `t2.micro` (Free tier)
   - **Key pair**: Create or use an existing key pair
   - **Network settings**:
     - Allow **SSH** (port 22)
     - Add **Custom TCP** for ports `3306`, `5432`, `27017` (for temporary access only)
   - **Storage**: 8 GB is enough
4. Click **Launch Instance**
5. Connect via SSH:
   ```bash
   ssh -i your-key.pem ubuntu@your-ec2-public-ip
   ```

## 🛠️ Step 2: Update & Install Tools
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install wget curl gnupg2 software-properties-common -y
```

## 🐬 Step 3: Install & Configure MySQL

### Install MySQL
```bash
sudo apt install mysql-server -y
```

### Secure MySQL
```bash
sudo mysql_secure_installation
```

### Create DB & User
```bash
sudo mysql -u root -p
```
Inside MySQL shell:
```sql
CREATE DATABASE testdb_mysql;
CREATE USER 'testuser'@'localhost' IDENTIFIED BY 'StrongPass123!';
GRANT ALL PRIVILEGES ON testdb_mysql.* TO 'testuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Restrict Remote Access
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
# Change bind-address = 127.0.0.1
sudo systemctl restart mysql
```

## 🐘 Step 4: Install & Configure PostgreSQL

### Install PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib -y
```

### Create DB & User
```bash
sudo -i -u postgres
psql
```
Inside PostgreSQL shell:
```sql
CREATE DATABASE testdb_pg;
CREATE USER testuser WITH ENCRYPTED PASSWORD 'StrongPass123!';
GRANT ALL PRIVILEGES ON DATABASE testdb_pg TO testuser;
\q
exit
```

### Restrict Remote Access
```bash
sudo nano /etc/postgresql/*/main/postgresql.conf
# listen_addresses = 'localhost'

sudo nano /etc/postgresql/*/main/pg_hba.conf
# local   all   all   peer

sudo systemctl restart postgresql
```

## 🍃 Step 5: Install & Configure MongoDB

### Install MongoDB 8.0
```bash
sudo apt-get install gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

### Create DB & User
```bash
mongosh
```
Inside Mongo shell:
```javascript
use testdb_mongo
db.createUser({ user: "testuser", pwd: "StrongPass123!", roles: ["readWrite"] })
exit
```

### Restrict Remote Access
```bash
sudo nano /etc/mongod.conf
# net:
#   bindIp: 127.0.0.1

sudo systemctl restart mongod
```

## 📦 Deliverables

### Access Configuration Explanation
- **MySQL**: `bind-address` set to `127.0.0.1` in `mysqld.cnf`
- **PostgreSQL**: `listen_addresses = 'localhost'` in `postgresql.conf` and local-only rules in `pg_hba.conf`
- **MongoDB**: `bindIp` set to `127.0.0.1` in `mongod.conf`

### Database & User Details

| DBMS       | Database Name   | User      | Password         | Access |
|------------|------------------|-----------|------------------|--------|
| MySQL      | testdb_mysql     | testuser  | StrongPass123!   | Local  |
| PostgreSQL | testdb_pg        | testuser  | StrongPass123!   | Local  |
| MongoDB    | testdb_mongo     | testuser  | StrongPass123!   | Local  |
