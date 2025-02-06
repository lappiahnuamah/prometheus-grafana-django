# Monitoring a Django Application with Prometheus and Grafana

## Introduction

This documentation will guide you through setting up monitoring for a Django application using Prometheus and Grafana. The setup includes:

- Configuring Django to expose Prometheus metrics
- Setting up Prometheus to scrape metrics
- Integrating Grafana for visualization

## Prerequisites

Ensure you have the following installed:

- Docker & Docker Compose
- Django application
- Prometheus
- Grafana

---

## Step 1: Install and Configure Prometheus in Django

### 1. Install Prometheus Client for Django

In your Django project, install the Prometheus client:

```bash
pip install django-prometheus
```

### 2. Configure Django Settings

Modify `settings.py` to add `django_prometheus`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django_prometheus',  # Add this line
]
```

### 3. Update Middleware and Database Configuration

Modify `settings.py` to include Prometheus middleware:

```python
MIDDLEWARE = [
    'django_prometheus.middleware.PrometheusBeforeMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django_prometheus.middleware.PrometheusAfterMiddleware',
]
```

### 4. Update URLs to Expose Metrics

Modify `urls.py` to add the Prometheus metrics endpoint:

```python
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("metrics/", include("django_prometheus.urls")),  # Add this line
]
```

---

## Step 2: Setup Prometheus

### 1. Create `prometheus.yml` Configuration File

Create a `prometheus.yml` file to scrape Django metrics:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "django_app"
    scrape_interval: 5s
    metrics_path: "/metrics/"
    static_configs:
      - targets: ["<ip_address>:8000"]
    scheme: http

  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["node_exporter:9100"]
```

### 2. Create a Docker Compose File (`docker-compose.yml`)

Set up Django, Prometheus, and Grafana using Docker Compose:

```yaml
version: '3.8'

services:
  django_app:
    build: ./myproject
    container_name: django_app
    ports:
      - "8000:8000"
    networks:
      - monitoring
    restart: always

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    networks:
      - monitoring
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    networks:
      - monitoring

networks:
  monitoring:

volumes:
  grafana-data:
```

### 3. Start the Containers

Run the following command to start the setup:

```bash
docker-compose up -d
```

---

## Step 3: Set Up Grafana for Visualization

### 1. Access Grafana

Open your browser and go to:

➡️ [http://localhost:3000](http://localhost:3000) (or server IP: `http://your-server-ip:3000`)
---
![Grafana](1.png)
---
**Default credentials:**

- **Username:** `admin`
- **Password:** `admin` (change upon first login)

---
![change-password](2.png)
---

### 2. Add Prometheus as a Data Source

1. Go to **Configuration → Data Sources**
---
![data-sources](3.png)
---
2. Click **"Add data source"**
---
![add-data-sources](4.png)
---
3. Select **Prometheus**
4. Go to Prometheus Settings and set local Prometheus URL
6. Set the URL to:

```plaintext
http://prometheus:9090
```

5. Click **"Save & Test"**

### 3. Create a Dashboard

1. Go to **Dashboards → New Dashboard**
2. Click **"Add visualization"**
3. In the Query section, select **Prometheus** as the data source
4. Enter a query such as:

```plaintext
process_resident_memory_bytes
```

5. Click **Apply** and save the dashboard

---

## Step 4: Verify and Monitor Metrics

- Visit **Prometheus UI**: [http://localhost:9090](http://localhost:9090)
- Query Django metrics, e.g., `python_gc_objects_collected_total`
- Check **Grafana dashboard** for visualized metrics

---

## Conclusion

You have successfully set up monitoring for your Django application using Prometheus and Grafana. You can now:

✅ Track Django performance metrics  
✅ Visualize data in Grafana dashboards  

For advanced monitoring, consider adding **Alertmanager** and custom **PromQL queries**! 🚀🔥
```

This Markdown file can be used directly in GitHub, a wiki, or a README file for easy documentation. Let me know if you need any modifications! 🚀
