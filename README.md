# AWS VPC Networking Project

## Secure Multi-Tier Architecture 

---

## Project Overview

This project demonstrates the design and implementation of a secure, scalable, and highly available multi-tier architecture using AWS networking services.

The infrastructure simulates a real-world fintech environment, where strict security, controlled access, and layered architecture are critical.


---

## Architecture Summary

<img width="1536" height="1024" alt="Architectural Diagram" src="https://github.com/user-attachments/assets/625d3f7f-11c4-45bd-9d89-413b5532bf54" />


The architecture consists of three logical tiers:

* **Web Tier (Public Subnets)**
  Handles incoming HTTP/HTTPS requests

* **Application Tier (Private Subnets)**
  Processes business logic via a backend service

* **Database Tier (Private Subnets)**
  Stores and manages persistent data securely


---
## Skills Demonstrated

* AWS Networking (VPC, Subnets, Routing)
* Cloud Security Architecture
* Linux Server Configuration
* Infrastructure Design Thinking
* Debugging Network Connectivity
* Backend Deployment (Node.js)
* Database Setup (MySQL)

---

## Network Design

### VPC Configuration

| Component      | Value       |
| -------------- | ----------- |
| VPC Name       | Fintech-VPC |
| CIDR Block     | 10.0.0.0/16 |
| DNS Resolution | Enabled     |
| DNS Hostnames  | Enabled     |
<img width="974" height="420" alt="image" src="https://github.com/user-attachments/assets/6ead5ee2-13b5-4e40-8480-54aef34140f8" />

---

### Subnet Design

#### Public Subnets (Web Tier)

| Subnet | CIDR        | AZ         |
| ------ | ----------- | ---------- |
| Web-A  | 10.0.1.0/24 | us-east-1a |
| Web-B  | 10.0.2.0/24 | us-east-1b |

* Auto-assign Public IP: Enabled

---

#### Private App Subnets

| Subnet | CIDR        | AZ         |
| ------ | ----------- | ---------- |
| App-A  | 10.0.3.0/24 | us-east-1a |
| App-B  | 10.0.4.0/24 | us-east-1b |

* Public IP: Disabled

---

#### Private DB Subnets

| Subnet | CIDR        | AZ         |
| ------ | ----------- | ---------- |
| DB-A   | 10.0.5.0/24 | us-east-1a |
| DB-B   | 10.0.6.0/24 | us-east-1b |

* Public IP: Disabled
<img width="975" height="328" alt="image" src="https://github.com/user-attachments/assets/36aecf2f-f057-4de8-944d-fb52882a783b" />

---

## Internet & Routing

### Internet Gateway

* Attached to VPC
* Enables public internet access for Web Tier
<img width="975" height="205" alt="image" src="https://github.com/user-attachments/assets/a1af20bc-1ecf-40f2-8e0d-394dca272a8d" />

### NAT Gateway

* Deployed in Public Subnet (Web-A)
* Allows private subnets to access the internet securely
<img width="975" height="210" alt="image" src="https://github.com/user-attachments/assets/2c569fa9-0baa-41db-a421-02e8ed3aa728" />

---

### Route Tables

| Route Table    | مقصد      | Target           |
| -------------- | --------- | ---------------- |
| Public-RT      | 0.0.0.0/0 | Internet Gateway |
| Private-App-RT | 0.0.0.0/0 | NAT Gateway      |
| Private-DB-RT  | 0.0.0.0/0 | NAT Gateway      |
<img width="975" height="441" alt="image" src="https://github.com/user-attachments/assets/98ede3e6-a705-4544-99a3-3523d684ec58" />

<img width="975" height="440" alt="image" src="https://github.com/user-attachments/assets/5c37716e-9740-434d-9682-4584ae9f8f7b" />

<img width="975" height="449" alt="image" src="https://github.com/user-attachments/assets/0a8b88c8-c98d-48f1-9ac8-836a80da9aeb" />

---

## Security Configuration

### Web Tier Security Group

* HTTP (80) → 0.0.0.0/0
* HTTPS (443) → 0.0.0.0/0
* SSH (22) → 0.0.0.0/0 *(for demo purposes)*
* ICMP → 0.0.0.0/0

<img width="975" height="446" alt="image" src="https://github.com/user-attachments/assets/bde97a6d-c147-49ef-aa6e-ecb30debc4fa" />

---

### App Tier Security Group

* Custom TCP (3000) → Web Tier SG
* SSH → Bastion Host only
<img width="975" height="408" alt="image" src="https://github.com/user-attachments/assets/f30af8a6-9eb6-490e-910c-01933afcc497" />

---

### Database Security Group

* MySQL (3306) → App Tier SG only
<img width="975" height="416" alt="image" src="https://github.com/user-attachments/assets/ea8f6b9c-4ad2-48d5-8920-880c4d5c9638" />

---

### Bastion Host Security Group

* SSH (22) → Your local machine (restricted IP)

<img width="975" height="397" alt="image" src="https://github.com/user-attachments/assets/987abe90-7565-4e3e-bad3-de115dfff3cb" />
---

## EC2 Deployment

### Web Server (Public)

* Apache Web Server installed
* Serves frontend content

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

echo "<h1>Welcome to XFintech Web Tier</h1>" > /var/www/html/index.html
echo "<p>Powered By Chigozie Nwanze!</p>" >> /var/www/html/index.html

chown -R ec2-user:ec2-user /var/www/html
systemctl restart httpd
```
<img width="974" height="468" alt="image" src="https://github.com/user-attachments/assets/14801e97-d618-4330-9b25-73bed088c7bb" />

---

### Bastion Server (Public)
* Amazon Linux running on port 22
  <img width="974" height="465" alt="image" src="https://github.com/user-attachments/assets/4a3d84d4-07dc-4697-8e4c-4890d5dc3e08" />

---


### Application Server (Private)

* Node.js backend running on port 3000
* Managed using PM2

```bash
#!/bin/bash
yum update -y
curl -sL https://rpm.nodesource.com/setup_18.x | bash -
yum install -y nodejs

mkdir -p /home/ec2-user/app
cd /home/ec2-user/app

cat << 'EOF' > app.js
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200, {'Content-Type': 'application/json'});
    res.end(JSON.stringify({ message: 'Hello from App Tier!' }));
  } else {
    res.writeHead(404);
    res.end();
  }
});

server.listen(3000, '0.0.0.0');
EOF

npm install -g pm2
pm2 start app.js
pm2 startup
pm2 save
```
<img width="975" height="463" alt="image" src="https://github.com/user-attachments/assets/57ae6049-d0c4-497f-a9aa-a67c2231538d" />

---

### Database Server (Private)

* MySQL installed and secured

```bash
#!/bin/bash
yum update -y
yum install -y mysql-server

systemctl start mysqld
systemctl enable mysqld

TEMP_PASSWORD=$(grep 'temporary password' /var/log/mysqld.log | awk '{print $NF}')

mysql --connect-expired-password -u root -p"$TEMP_PASSWORD" <<EOF
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyPassword123!';
DELETE FROM mysql.user WHERE User='';
DROP DATABASE IF EXISTS test;
FLUSH PRIVILEGES;
EOF

sed -i 's/^bind-address.*/bind-address = 0.0.0.0/' /etc/my.cnf
systemctl restart mysqld
```
<img width="974" height="468" alt="image" src="https://github.com/user-attachments/assets/b00fdea3-d9e9-43de-926a-d81a8f515de7" />

---

## Connectivity Testing

### Test 1: Web → App

```bash
curl http://10.0.3.30:3000
```
{"message":"Hello from App Tier!","status":"success"}
<img width="916" height="336" alt="image" src="https://github.com/user-attachments/assets/abbb8dc2-fd68-4b9d-88e3-e57c590c641f" />

---

### Test 2: App → Database

Using Bastion Host:

```bash
ssh -i key.pem ec2-user@<bastion-ip>
ssh -i key.pem ec2-user@<app-private-ip>
<img width="928" height="505" alt="image" src="https://github.com/user-attachments/assets/1391082f-9dfd-4389-ba00-eba5e0ca71b9" />

<img width="975" height="288" alt="image" src="https://github.com/user-attachments/assets/6cb62190-bf5e-4157-a166-2ca55b255485" />

nc -zv <db-private-ip> 3306
```
<img width="839" height="177" alt="image" src="https://github.com/user-attachments/assets/e7c4f733-ec19-4caa-a998-1348b7f9d369" />

---


## Author

**Chigozie Nwanze**
(Cloud Engineer)
