# Project Documentation

## Overview

This repository contains a minimal Flask application deployed behind Nginx and Gunicorn. The project is designed to run as a Dockerized web service and exposes a simple health endpoint and homepage.

The application is a good starting point for a Python web service that can be deployed to platforms such as Passenger, Docker, or a Linux VM using Nginx as a reverse proxy.

## Repository Structure

```text
.
├── app/
│   ├── __init__.py
│   ├── app.py
│   └── static/
├── Dockerfile
├── README.md
├── passenger_wsgi.py
├── requirements.txt
├── webapp.conf
└── DOCUMENTATION.md
```

## Application Components

### Flask App
The app is created in `app/app.py` using Flask.

```python
from flask import Flask

def create_app():
    app = Flask(__name__)

    @app.route("/")
    def index():
        return "Hello from Python on Passenger + Nginx!"

    @app.route("/health")
    def health():
        return "ok", 200

    return app
```

This application exposes:
- `/` - returns a welcome message.
- `/health` - returns `ok` with HTTP 200 for health checks.

### WSGI Entry Point
`passenger_wsgi.py` is the entry point used by the application server:

```python
from app.app import create_app
application = create_app()
```

This allows the application to be served by Gunicorn or similar WSGI servers.

### Nginx Configuration
`webapp.conf` routes incoming requests to the Flask app running on port `8000`.

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /home/app/webapp/app/static/;
    }
}
```

This means:
- Nginx listens on port `80`.
- Requests are proxied to the Flask app on `127.0.0.1:8000`.
- Static files are served directly from `app/static/`.

## Dependencies

The project dependencies are defined in `requirements.txt`:

```txt
Flask==3.0.0
gunicorn==21.2.0
```

These packages provide:
- Flask: web application framework
- Gunicorn: Python WSGI HTTP server for production deployment

## Docker Setup

The `Dockerfile` builds a Python 3.12 slim image and configures the app to run with Nginx and Gunicorn.

### Key parts of the Dockerfile

```dockerfile
FROM python:3.12-slim

RUN apt-get update && apt-get install -y nginx curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /home/app/webapp

COPY requirements.txt .
COPY passenger_wsgi.py .
COPY app/ ./app/
COPY webapp.conf /etc/nginx/conf.d/webapp.conf

RUN pip install --no-cache-dir -r requirements.txt
RUN rm /etc/nginx/sites-enabled/default

EXPOSE 80

CMD service nginx start && gunicorn -w 4 -b 0.0.0.0:8000 passenger_wsgi:application
```

### How the container works
- Installs Python, Nginx, and curl.
- Copies the Flask app and config files into the image.
- Installs required Python packages.
- Starts Nginx and Gunicorn when the container launches.
- Exposes port `80` externally.

## Local Development

### Prerequisites
- Python 3.10+
- pip

### Install dependencies

```bash
pip install -r requirements.txt
```

### Run the app locally

```bash
python -m flask --app app.app:create_app run
```

Or, if you want to use the app factory directly:

```bash
export FLASK_APP=app.app:create_app
flask run
```

### Run with Gunicorn

```bash
gunicorn -w 4 -b 0.0.0.0:8000 passenger_wsgi:application
```

## Health Check

The service includes a basic health endpoint for monitoring:

```bash
curl http://localhost:8000/health
```

Expected result:

```text
ok
```

## Deployment Notes

### Typical production flow
1. Build the Docker image.
2. Start the container.
3. Nginx listens on port 80.
4. Requests are proxied to Gunicorn on port 8000.
5. Flask serves the app from `passenger_wsgi.py`.

### Example Docker build

```bash
docker build -t python-passenger-app .
docker run -p 80:80 python-passenger-app
```

## Future Improvements

This project is intentionally simple and can be extended with:
- more route handlers and APIs
- database integration
- environment-based configuration
- structured logging
- tests and CI automation
- deployment to cloud or container platforms

## Summary

This repository demonstrates a lightweight Python Flask application deployed behind Nginx and Gunicorn. It is suitable as a starting template for web applications that need a simple production-ready web stack with Docker support.

---

For additional project details, see `README.md`.
