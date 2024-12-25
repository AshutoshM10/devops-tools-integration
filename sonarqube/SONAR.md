# Setting up SonarQube on Ubuntu

This guide provides detailed steps to install and configure SonarQube on an Ubuntu server.

---

## Prerequisites

1. **System Requirements**:
   - Minimum 2 GB RAM.
   - At least 1 CPU core (recommended: t2.medium or higher for AWS EC2).

2. **Java Installation**:
   - SonarQube requires Java 11 or 17. Ensure it's installed:
     ```bash
     sudo apt update
     sudo apt install openjdk-17-jdk -y
     ```
   - Verify the Java version:
     ```bash
     java -version
     ```

3. **Database**:
   - SonarQube requires PostgreSQL as its database.
   - Install PostgreSQL:
     ```bash
     sudo apt update
     sudo apt install postgresql postgresql-contrib -y
     ```
   - Configure PostgreSQL:
     ```bash
     sudo -u postgres psql
     CREATE DATABASE sonarqube;
     CREATE USER sonar WITH ENCRYPTED PASSWORD 'sonar_password';
     GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
     \q
     ```

---

## Step 1: Download and Extract SonarQube

1. Download SonarQube:
   ```bash
   wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.2.0.77687.zip
   ```

2. Install Unzip and extract the files:
   ```bash
   sudo apt install unzip -y
   unzip sonarqube-10.2.0.77687.zip
   sudo mv sonarqube-10.2.0.77687 /opt/sonarqube
   ```

---

## Step 2: Configure SonarQube

1. **Modify Configuration File**:
   ```bash
   sudo nano /opt/sonarqube/conf/sonar.properties
   ```
   Update the following:
   ```properties
   sonar.jdbc.username=sonar
   sonar.jdbc.password=sonar_password
   sonar.jdbc.url=jdbc:postgresql://127.0.0.1:5432/sonarqube
   ```

2. **Elasticsearch Memory Configuration**:
   Edit the `sonar.sh` file:
   ```bash
   sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
   ```
   Increase the heap size if necessary (e.g., `-Xms1G -Xmx2G`).

---

## Step 3: Create a Systemd Service for SonarQube

1. **Create Service File**:
   ```bash
   sudo nano /etc/systemd/system/sonarqube.service
   ```
   Add the following:
   ```ini
   [Unit]
   Description=SonarQube service
   After=syslog.target network.target

   [Service]
   Type=forking
   ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
   ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
   User=sonarqube
   Group=sonarqube
   Restart=always
   LimitNOFILE=65536

   [Install]
   WantedBy=multi-user.target
   ```

2. **Create User and Set Permissions**:
   ```bash
   sudo useradd -r -s /bin/false sonarqube
   sudo chown -R sonarqube:sonarqube /opt/sonarqube
   ```

3. **Reload and Start the Service**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start sonarqube
   sudo systemctl enable sonarqube
   ```

---

## Step 4: Access SonarQube

1. **Open Ports**:
   Ensure port 9000 is open in your firewall or security groups.

2. **Access the Web Interface**:
   Navigate to `http://<your-server-ip>:9000`.

3. **Login Credentials**:
   - Username: `admin`
   - Password: `admin`

4. **Initial Configuration**:
   - Change the default admin password.
   - Configure projects and quality profiles as needed.

---

## Troubleshooting

- **ElasticSearch Errors**:
  - Check memory allocation: Ensure you have enough RAM allocated to the server.
  - Logs: Examine `/opt/sonarqube/logs/sonar.log` for error details.

- **Port Not Accessible**:
  - Verify firewall rules or security groups to ensure port 9000 is open.

---

## Conclusion
SonarQube is now up and running on your Ubuntu server. You can integrate it with Jenkins or any CI/CD tool for code quality analysis and reporting.
