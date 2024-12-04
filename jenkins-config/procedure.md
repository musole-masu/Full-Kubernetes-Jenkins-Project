# Setting Up A Jenkins Server
**Outcomes**: A Jenkins server with the necessary plugins to automate the testing, building, and deployment of a Node.js application to a Kubernetes cluster. 

**Prerequisites**: 1 virtual machines running Ubuntu 20.04 LTS (or the latest release), with a minimum of 2 CPUs, 4 GB of RAM, and 50 GB of storage.

## Jenkins Installation

### 1. Installation of Java

Jenkins requires Java to run

```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)

```

### 2. Long Term Support release

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf upgrade
# Add required dependencies for the jenkins package
sudo dnf install fontconfig java-17-openjdk
sudo dnf install jenkins
sudo systemctl daemon-reload

```

### 3. Start Jenkins

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

```