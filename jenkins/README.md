# Jenkins Setup Guide

This guide will walk you through the process of setting up Jenkins on an Ubuntu server, configuring it for CI/CD pipelines, and troubleshooting common issues.

## Prerequisites

Before you begin, ensure you have:

- An Ubuntu server (VM or cloud-based).
- Sudo or root access.
- OpenJDK 11 installed (or a compatible Java version).

## Installation Steps

### Step 1: Install Java

Jenkins requires Java to run. Install OpenJDK 17:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
```

Verify the Java version. It should display Java 17.

### Step 2: Add the Jenkins Repository

Add the Jenkins Debian package repository and the GPG key:

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee "/usr/share/keyrings/jenkins-keyring.asc" > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Step 3: Install Jenkins

Update the package list and install Jenkins:

```bash
sudo apt update
sudo apt install jenkins -y
```

### Step 4: Start Jenkins Service

Start and enable the Jenkins service:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Check the service status:

```bash
sudo systemctl status jenkins
```

### Step 5: Open Firewall Port (if applicable)

If you are using UFW, allow Jenkins through the firewall:

```bash
sudo ufw allow 8080
sudo ufw reload
```

### Step 6: Access Jenkins

- Open a web browser and navigate to `http://<your-server-ip>:8080`.
- Locate the initial admin password by running:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Copy the password and paste it into the Jenkins setup wizard.

### Step 7: Complete Jenkins Setup

- Install suggested plugins or customize the plugins based on your needs.
- Create the first admin user.
- Jenkins is now ready to use!

---

## Troubleshooting Common Issues

### Jenkins Fails to Start

#### Check Logs

Run the following command to inspect Jenkins logs:

```bash
sudo journalctl -xeu jenkins.service
sudo cat /var/log/jenkins/jenkins.log
```

#### Java Version Issue

Ensure Java 17 is installed:

```bash
java -version
```

If not, install or switch to Java 17:

```bash
sudo apt install openjdk-17-jdk -y
sudo update-alternatives --config java
```
### Set the JAVA_HOME path
```
JAVA_HOME=/usr/lib/jvm/java-1.17.0-openjdk-amd64
```
#### Port Conflict

Check if port 8080 is in use:

```bash
sudo netstat -tuln | grep 8080
```

Modify the port in `/etc/default/jenkins`:

```bash
sudo nano /etc/default/jenkins
# Update HTTP_PORT=8080 to another port, e.g., 8081
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

#### Permissions Issue

Ensure Jenkins has the necessary permissions:

```bash
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo chown -R jenkins:jenkins /var/log/jenkins
sudo chown -R jenkins:jenkins /var/cache/jenkins
```

---

## Additional Configurations

### Plugins

Install recommended plugins or others based on your project needs:

- **Git** for version control.
- **Pipeline** for creating CI/CD workflows.
- **SonarQube** for code quality checks.
- **OWASP Dependency-Check** for security scans.


---

## References

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Plugins Index](https://plugins.jenkins.io/)
