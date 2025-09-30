# BunkerWeb

## Introduction

BunkerWeb is a next-generation, open-source Web Application Firewall (WAF) designed to protect your web applications and services from various cyber threats. Built with modern security practices in mind, BunkerWeb provides comprehensive protection against common web vulnerabilities while maintaining high performance and ease of deployment.

Unlike traditional WAFs that can be complex to configure and maintain, BunkerWeb offers an intuitive approach to web security with automated threat detection, real-time protection, and seamless integration into modern DevOps workflows. Whether you're running a simple website or a complex microservices architecture, BunkerWeb adapts to your needs while providing enterprise-grade security.

## Key Concepts

### Core Architecture

BunkerWeb operates as a reverse proxy that sits between your web applications and the internet, analyzing all incoming traffic before it reaches your backend services. This positioning allows it to:

- **Filter malicious requests** before they reach your application
- **Block known attack patterns** using signature-based detection
- **Rate limit suspicious behavior** to prevent abuse
- **Provide SSL/TLS termination** with automatic certificate management

### Security Modules

BunkerWeb's protection capabilities are organized into specialized modules:

#### **Anti-Bot Protection**
- Detects and blocks automated bot traffic
- Implements CAPTCHA challenges for suspicious requests
- Uses behavioral analysis to distinguish between legitimate users and bots
- Supports whitelist/blacklist management for known good/bad bots

#### **DDoS Protection**
- Rate limiting based on various criteria (IP, user agent, etc.)
- Connection limiting to prevent resource exhaustion
- Automatic temporary blocking of abusive sources
- Distributed attack mitigation strategies

#### **Web Application Security**
- OWASP Top 10 protection out of the box
- SQL injection prevention
- Cross-Site Scripting (XSS) filtering
- Command injection detection
- Path traversal protection

#### **Reputation-Based Filtering**
- Integration with threat intelligence feeds
- IP reputation scoring
- Geolocation-based blocking
- Real-time blacklist updates

### Deployment Models

BunkerWeb supports multiple deployment scenarios to fit different infrastructure needs:

#### **Standalone Mode**
- Single instance deployment
- Ideal for small to medium applications
- Simple configuration and management
- Direct integration with existing web servers

#### **Cluster Mode**
- Multi-instance deployment for high availability
- Load balancing across multiple BunkerWeb instances
- Shared configuration and threat intelligence
- Horizontal scaling capabilities

#### **Container Integration**
- Native Docker support
- Kubernetes deployment options
- Helm charts for easy installation
- Integration with container orchestration platforms

#### **Cloud-Native Deployment**
- Support for major cloud providers
- Auto-scaling capabilities
- Integration with cloud security services
- Managed service options

### Configuration Philosophy

BunkerWeb follows a "secure by default" approach:

- **Zero-configuration security**: Essential protections are enabled automatically
- **Gradual customization**: Advanced features can be enabled as needed
- **Policy-based management**: Security rules are organized into logical policies
- **Environment-specific configs**: Different settings for development, staging, and production

### Real-Time Monitoring

The platform provides comprehensive visibility into security events:

- **Live traffic analysis** with detailed request inspection
- **Security event logging** with customizable alert thresholds
- **Performance metrics** to monitor impact on application response times
- **Threat intelligence reporting** with actionable insights

### Integration Ecosystem

BunkerWeb integrates seamlessly with existing tools and workflows:

- **CI/CD pipelines** for automated security testing
- **SIEM systems** for centralized log management
- **Monitoring platforms** like Prometheus and Grafana
- **Authentication providers** for user-based policies
- **Content Delivery Networks** for global protection

## Benefits

- **Comprehensive Protection**: Guards against the full spectrum of web-based attacks
- **Easy Deployment**: Multiple installation methods with minimal configuration required
- **High Performance**: Optimized for low latency and high throughput
- **Cost Effective**: Open-source solution with enterprise features
- **Scalable**: Grows with your infrastructure needs
- **Community Driven**: Active development and support community

BunkerWeb represents a modern approach to web application security, combining the robustness of enterprise-grade protection with the flexibility and transparency of open-source software.


## Quick start
### Install
- **Add Helm Repo**
![addhelm](/imgs/addHelmRepo.png)
- **Edit the Value file**
![valueFile](/imgs/valuesFile.png)
- **Install Bunker with edited Value file**
![InstallBunker](/imgs/installBunker.png)
- **Expose Bunker UI**
![exBunkerUI](/imgs/exposedBunkerUI.png)
- **Access the UI with configured password**
![BunkerUI](/imgs/BunkerUI.png)
## Installation

BunkerWeb offers multiple installation methods to accommodate different infrastructure setups and deployment preferences. Choose the method that best fits your environment and requirements.

### Prerequisites

Before installing BunkerWeb, ensure your system meets the following requirements:

- **Operating System**: Linux (Ubuntu 20.04+, Debian 11+, CentOS 8+, RHEL 8+)
- **Memory**: Minimum 1GB RAM (2GB+ recommended for production)
- **CPU**: 1+ cores (2+ cores recommended for production)
- **Storage**: 2GB+ available disk space
- **Network**: Internet connectivity for updates and threat intelligence feeds

### Docker Installation (Recommended)

Docker is the fastest and most reliable way to get BunkerWeb running:

#### Quick Start with Docker
```bash
# Pull the latest BunkerWeb image
docker pull bunkerity/bunkerweb:latest

# Run BunkerWeb with basic configuration
docker run -d \
  --name bunkerweb \
  -p 80:8080 \
  -p 443:8443 \
  -e SERVER_NAME=example.com \
  -e USE_REVERSE_PROXY=yes \
  -e REVERSE_PROXY_URL=/ \
  -e REVERSE_PROXY_HOST=http://backend:8080 \
  bunkerity/bunkerweb:latest
```

#### Docker Compose Setup
Create a `docker-compose.yml` file for a complete setup:

```yaml
version: '3.8'

services:
  bunkerweb:
    image: bunkerity/bunkerweb:latest
    container_name: bunkerweb
    ports:
      - "80:8080"
      - "443:8443"
    environment:
      - SERVER_NAME=example.com
      - USE_REVERSE_PROXY=yes
      - REVERSE_PROXY_URL=/
      - REVERSE_PROXY_HOST=http://backend:8080
      - AUTO_LETS_ENCRYPT=yes
      - USE_MODSECURITY=yes
      - USE_ANTIBOT=yes
    volumes:
      - bw_data:/data
    networks:
      - bunkerweb-net

  backend:
    image: nginx:alpine
    container_name: backend
    networks:
      - bunkerweb-net

volumes:
  bw_data:

networks:
  bunkerweb-net:
    driver: bridge
```

Deploy with:
```bash
docker-compose up -d
```

### Kubernetes Installation

#### Using Helm Charts
```bash
# Add BunkerWeb Helm repository
helm repo add bunkerweb https://charts.bunkerweb.io
helm repo update

# Install BunkerWeb
helm install bunkerweb bunkerweb/bunkerweb \
  --set config.serverName="example.com" \
  --set config.useReverseProxy="yes" \
  --set config.reverseProxyUrl="/" \
  --set config.reverseProxyHost="http://backend-service:80"
```

#### Manual Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bunkerweb
spec:
  replicas: 2
  selector:
    matchLabels:
      app: bunkerweb
  template:
    metadata:
      labels:
        app: bunkerweb
    spec:
      containers:
      - name: bunkerweb
        image: bunkerity/bunkerweb:latest
        ports:
        - containerPort: 8080
        - containerPort: 8443
        env:
        - name: SERVER_NAME
          value: "example.com"
        - name: USE_REVERSE_PROXY
          value: "yes"
        - name: REVERSE_PROXY_URL
          value: "/"
        - name: REVERSE_PROXY_HOST
          value: "http://backend-service:80"
```

### Linux Package Installation

#### Ubuntu/Debian
```bash
# Add BunkerWeb repository
curl -fsSL https://repo.bunkerweb.io/gpg.key | sudo apt-key add -
echo "deb https://repo.bunkerweb.io/debian stable main" | sudo tee /etc/apt/sources.list.d/bunkerweb.list

# Update package list and install
sudo apt update
sudo apt install bunkerweb

# Start and enable the service
sudo systemctl enable bunkerweb
sudo systemctl start bunkerweb
```

#### CentOS/RHEL/Fedora
```bash
# Add BunkerWeb repository
sudo tee /etc/yum.repos.d/bunkerweb.repo << EOF
[bunkerweb]
name=BunkerWeb Repository
baseurl=https://repo.bunkerweb.io/rpm/
enabled=1
gpgcheck=1
gpgkey=https://repo.bunkerweb.io/gpg.key
EOF

# Install BunkerWeb
sudo dnf install bunkerweb  # or 'yum install bunkerweb' for older systems

# Start and enable the service
sudo systemctl enable bunkerweb
sudo systemctl start bunkerweb
```

### Source Installation

For advanced users who want to build from source:

```bash
# Clone the repository
git clone https://github.com/bunkerity/bunkerweb.git
cd bunkerweb

# Install dependencies
sudo apt install python3 python3-pip nginx-dev  # Ubuntu/Debian
# or
sudo dnf install python3 python3-pip nginx-devel  # CentOS/RHEL

# Install Python dependencies
pip3 install -r requirements.txt

# Build and install
sudo python3 setup.py install

# Configure systemd service
sudo cp scripts/bunkerweb.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable bunkerweb
sudo systemctl start bunkerweb
```

### Cloud Provider Installations

#### AWS
Deploy using CloudFormation template:
```bash
aws cloudformation create-stack \
  --stack-name bunkerweb \
  --template-url https://templates.bunkerweb.io/aws/bunkerweb.yaml \
  --parameters ParameterKey=ServerName,ParameterValue=example.com
```

#### Google Cloud Platform
```bash
# Deploy using Google Cloud Deployment Manager
gcloud deployment-manager deployments create bunkerweb \
  --config https://templates.bunkerweb.io/gcp/bunkerweb.yaml
```

#### Azure
```bash
# Deploy using ARM template
az deployment group create \
  --resource-group myResourceGroup \
  --template-uri https://templates.bunkerweb.io/azure/bunkerweb.json
```

### Post-Installation Configuration

After installation, configure BunkerWeb by editing the main configuration file:

```bash
# For package installations
sudo nano /etc/bunkerweb/variables.env

# For Docker installations, use environment variables or volume-mounted config
```

Basic configuration example:
```bash
SERVER_NAME=example.com
USE_REVERSE_PROXY=yes
REVERSE_PROXY_URL=/
REVERSE_PROXY_HOST=http://127.0.0.1:8080
AUTO_LETS_ENCRYPT=yes
USE_MODSECURITY=yes
USE_ANTIBOT=yes
USE_WHITELIST=yes
WHITELIST_IP=127.0.0.1/8 10.0.0.0/8
```

### Verification

Verify your installation:

```bash
# Check service status
sudo systemctl status bunkerweb

# Test configuration
curl -H "Host: example.com" http://localhost

# View logs
sudo journalctl -u bunkerweb -f
```

### Next Steps

1. **Configure your backend**: Point BunkerWeb to your application servers
2. **Set up SSL certificates**: Enable HTTPS with Let's Encrypt or custom certificates
3. **Customize security rules**: Adjust protection levels based on your needs
4. **Monitor and tune**: Use the web interface to monitor traffic and adjust settings
5. **Set up backups**: Configure regular backups of your BunkerWeb configuration

For detailed configuration options and advanced setups, refer to the [Configuration Guide](https://docs.bunkerweb.io/1.6.4/configuration/) in the official documentation.