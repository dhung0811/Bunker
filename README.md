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
```
helm repo add bunkerweb https://repo.bunkerweb.io/charts
helm repo update
```

![addhelm](/imgs/addHelmRepo.png)

- **Edit the Value file**

![valueFile](/imgs/valuesFile.png)

- **Install Bunker with edited Value file**

``` helm install <release name> <chart reference> -n <namespace> -f <value file> ```

![InstallBunker](/imgs/installBunker.png)

- **Expose Bunker UI**

![exBunkerUI](/imgs/exposedBunkerUI.png)

- **Access the UI with configured password**

![BunkerUI](/imgs/BunkerUI.png)

### Configure
#### Create Namespace and Ingress Class
``` kubectl create namespace bunkerweb-system ```

Create Ingress Class with yaml:
```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: bunkerweb
spec:
  controller: bunkerweb.io/ingress-controller
```
Apply yaml file

``` kubectl apply -f bunkerweb-ingressclass.yaml ```

#### Write an example Service

``` 
apiVersion: v1
kind: Service
metadata:
  name: shop-svc
  namespace: default
spec:
  selector:
    app: shop
  ports:
  - port: 80
    targetPort: 8080
```

#### Create a BunkerWeb Ingress
```
# shop-bunkerweb-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shop-bw
  namespace: default
  annotations:
    # --- BunkerWeb features (per Ingress) ---
    bunkerweb.io/waf: "on"
    bunkerweb.io/waf-crs: "on"
    bunkerweb.io/waf-paranoia-level: "1"
    bunkerweb.io/rate-limit: "100r/s"
    bunkerweb.io/rate-burst: "50"
    bunkerweb.io/client-max-body-size: "10m"
spec:
  ingressClassName: bunkerweb
  rules:
  - host: bunkerweb.test
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: shop-svc
            port:
              number: 80
```

Use annotations to config and use [features](https://docs.bunkerweb.io/1.6.4/features/) of Bunkerweb.

Then apply ingress

``` kubectl apply -f shop-bunkerweb-ingress.yaml ```

Results

![Created](/imgs/ingCreated.png)

![Settings](/imgs/setting.png)


