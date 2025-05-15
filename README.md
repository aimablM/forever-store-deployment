# Forever Store - Full Stack E-Commerce Platform

![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge&logo=open-source-initiative&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white)

A containerized, cloud-deployed e-commerce application showcasing modern DevOps practices with Docker, AWS, and automated infrastructure. This project demonstrates the full lifecycle of forking an open-source application, containerizing its components, and deploying to production with proper cloud engineering practices.

**Live Application**: [forever.aimablem.dev](https://forever.aimablem.dev)  
**API Endpoint**: [api.forever.aimablem.dev](https://api.forever.aimablem.dev)

> This deployment is part of a larger ecosystem of containerized applications running on a single AWS instance with domain isolation, showcasing efficient resource utilization while maintaining service boundaries.

## Project Highlights

- **Containerized Microservices**: Separate containers for frontend, backend, and database
- **Docker Compose Orchestration**: Multi-container deployment with proper networking
- **AWS Cloud Deployment**: Running on EC2 with proper security groups and networking
- **HTTPS Security**: SSL/TLS with Let's Encrypt certificates via Certbot
- **Reverse Proxy**: NGINX configuration with subdomain routing
- **Custom Domain**: AWS Route 53 integration with proper DNS records

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Technology Stack](#technology-stack)
- [Infrastructure Setup](#infrastructure-setup)
- [Docker Configuration](#docker-configuration)
- [Application Components](#application-components)
- [Deployment Process](#deployment-process)
- [Security Implementation](#security-implementation)
- [Monitoring & Maintenance](#monitoring--maintenance)
- [Future Improvements](#future-improvements)
- [Getting Started](#getting-started)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Architecture Overview

The Forever Store deployment follows a modern containerized architecture pattern with proper isolation and resource management:

```
                          ┌─────────────────┐
                          │  AWS Route 53   │
                          │   DNS Records   │
                          └────────┬────────┘
                                   │
                                   ▼
┌───────────────────────── AWS EC2 Instance ─────────────────────────┐
│                                                                     │
│  ┌─────────────────┐    ┌─────────────────────────────────────┐    │
│  │     NGINX       │    │      Docker Compose Network         │    │
│  │  Reverse Proxy  │───▶│                                     │    │
│  └─────────────────┘    │  ┌─────────────┐  ┌─────────────┐   │    │
│                         │  │  Frontend   │  │   Backend   │   │    │
│                         │  │  Container  │  │  Container  │   │    │
│  ┌─────────────────┐    │  └─────────────┘  └──────┬──────┘   │    │
│  │  Let's Encrypt  │    │                          │          │    │
│  │  Certificates   │    │                    ┌─────▼──────┐   │    │
│  └─────────────────┘    │                    │  MongoDB   │   │    │
│                         │                    │ Container  │   │    │
│                         │                    └────────────┘   │    │
│                         └─────────────────────────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

The architecture provides:
- Clear separation of concerns with each service in its own container
- Internal networking between containers via Docker Compose
- Centralized SSL handling via NGINX
- External access only through designated ports (80/443)

## Technology Stack

### Frontend
- **React**: JavaScript framework for the UI
- **HTML/CSS**: Responsive design with Tailwind CSS
- **Axios**: HTTP client for API communication
- **React Router**: Client-side routing

### Backend
- **Node.js**: JavaScript runtime
- **Express.js**: Web framework for REST API
- **MongoDB**: NoSQL database for persistent storage
- **JWT**: Authentication token management
- **Multer**: File upload handling
- **Cloudinary**: Cloud image storage

### DevOps & Infrastructure
- **Docker**: Container platform
- **Docker Compose**: Multi-container orchestration
- **NGINX**: Reverse proxy and HTTPS termination
- **Let's Encrypt/Certbot**: SSL certificate automation
- **AWS EC2**: Virtual server hosting
- **AWS Route 53**: DNS management

## Infrastructure Setup

The Forever Store is deployed on AWS infrastructure that hosts multiple containerized applications, each with their own subdomain and isolation:

### EC2 Configuration
- **Instance Type**: t2.micro (part of AWS Free Tier)
- **Operating System**: Ubuntu 22.04 LTS
- **Security Groups**:
  - HTTP (Port 80) - For initial web access and Let's Encrypt verification
  - HTTPS (Port 443) - For secure web traffic
  - SSH (Port 22) - For secure administrative access

### Domain Management
- **Primary Domain**: aimablem.dev (managed via Route 53)
- **Subdomains**:
  - **forever.aimablem.dev**: Frontend application
  - **api.forever.aimablem.dev**: Backend API
- **DNS Records**:
  - A Records pointing to EC2 public IP
  - CNAME Records for subdomains

### NGINX Configuration

```nginx
server {
    listen 80;
    server_name forever.aimablem.dev api.forever.aimablem.dev;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name forever.aimablem.dev;

    ssl_certificate /etc/letsencrypt/live/forever.aimablem.dev/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/forever.aimablem.dev/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:3100;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 443 ssl;
    server_name api.forever.aimablem.dev;

    ssl_certificate /etc/letsencrypt/live/api.forever.aimablem.dev/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.forever.aimablem.dev/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:4100;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Docker Configuration

The application is containerized using Docker, with services defined and managed through Docker Compose:

### Docker Compose Configuration

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      args:
        VITE_BACKEND_URL: https://api.forever.aimablem.dev
    ports:
      - "3100:80"
    depends_on:
      - backend
    networks:
      - appnet

  backend:
    build: ./backend
    ports:
      - "4100:4000"
    depends_on:
      - mongodb
    environment:
      - JWT_SECRET=
      - ADMIN_EMAIL=
      - ADMIN_PASSWORD=
      - MONGODB_URI=
      - CLOUDINARY_API_KEY=
      - CLOUDINARY_SECRET_KEY=
      - CLOUDINARY_NAME=
      - STRIPE_SECRET_KEY=
    networks:
      - appnet

  mongodb:
    image: mongo:latest
    restart: always
    volumes:
      - forever_mongo_data:/data/db
    networks:
      - appnet

volumes:
  forever_mongo_data:

networks:
  appnet:
    driver: bridge
```

### Frontend Dockerfile

```dockerfile
# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Accept env var from build args
ENV VITE_BACKEND_URL=https://api.forever.aimablem.dev

# Install dependencies first (optimized layer caching)
COPY package.json package-lock.json* pnpm-lock.yaml* yarn.lock* ./
RUN npm install

# Copy all source files
COPY . .

# Build the app
RUN npm run build

# Production stage
FROM nginx:stable-alpine

# Set working directory
WORKDIR /usr/share/nginx/html

# Remove default nginx static assets
RUN rm -rf ./*

# Copy built assets from builder stage
COPY --from=builder /app/dist .

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Backend Dockerfile

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:20

# Set the working directory inside the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install app dependencies
RUN npm install --production

# Copy the rest of the application code
COPY . .

# Expose the port the app runs on
EXPOSE 4000

# Command to run the app
CMD ["npm", "start"]
```

## Application Components

The Forever Store consists of several key components that work together to create a complete e-commerce experience:

### Admin Panel
- Product management (add, edit, remove)
- Order tracking and status updates
- User management

### Customer Frontend
- Product browsing and filtering
- Cart management
- Checkout process
- Order history

### Backend API
- User authentication
- Product catalog management
- Order processing
- Payment integration (Stripe and Razorpay)
- Image upload and management via Cloudinary

## Deployment Process

The deployment process follows these steps:

1. **Fork and Clone**: The open-source e-commerce repository was forked and cloned for customization
2. **Containerization**: Added Dockerfiles and Docker Compose configuration
3. **Infrastructure Setup**: Provisioned EC2 instance and configured security groups
4. **DNS Configuration**: Set up subdomains in AWS Route 53
5. **SSL Certificate Setup**: Used Certbot to obtain and configure Let's Encrypt certificates
6. **NGINX Configuration**: Set up reverse proxy with proper routing
7. **Container Deployment**: Built and deployed the Docker containers
8. **Testing and Validation**: Verified the full application functionality

Future deployments will be automated through CI/CD pipeline integration.

## Security Implementation

Security is implemented at multiple layers:

### Network Security
- **Security Groups**: AWS EC2 security groups restrict access to only necessary ports
- **HTTPS Encryption**: All traffic is encrypted via SSL/TLS
- **HTTP to HTTPS Redirection**: All HTTP requests are automatically redirected to HTTPS

### Application Security
- **JWT Authentication**: Secure token-based authentication for API access
- **Role-Based Access Control**: Admin vs. customer access separation
- **Environment Variables**: Sensitive configuration stored in environment variables
- **Container Isolation**: Services run in isolated containers

### Data Security
- **MongoDB Authentication**: Database access restricted by credentials
- **Persistent Volume**: Data stored in a Docker volume for durability
- **Regular Backups**: Scheduled database snapshots

## Monitoring & Maintenance

The application is configured with monitoring and maintenance tools:

### Health Checks
- Docker health checks for container monitoring
- AWS EC2 status checks
- Endpoint monitoring via cron jobs

### Logging
- Container logs collected and rotated
- Application-level logging for error tracking
- HTTP access logs via NGINX

### Maintenance Procedures
- Database backups using MongoDB dump
- Container updates with minimal downtime
- SSL certificate auto-renewal via Certbot

## Future Improvements

Several enhancements are planned for the near future:

1. **CI/CD Implementation**
   - GitHub Actions workflow for automated testing and deployment
   - Docker Hub integration for container image registry
   - Automated deployment triggered by commits to the main branch

2. **Monitoring Enhancements**
   - Prometheus and Grafana integration for metrics visualization
   - Alerting for critical system events
   - Log aggregation with ELK stack

3. **Infrastructure Optimization**
   - CloudFront integration for faster global content delivery
   - Auto-scaling configuration for handling traffic spikes
   - Load balancing for improved availability

4. **Feature Improvements**
   - Enhanced search functionality
   - User reviews and ratings
   - Wishlist functionality
   - Product recommendation engine

## Getting Started

To deploy this project in your own environment:

### Prerequisites

- AWS Account
- Domain name (for SSL setup)
- Docker and Docker Compose installed
- Git

### Deployment Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/YourUsername/forever-store.git
   cd forever-store
   ```

2. Configure environment variables:
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

3. Build and start the containers:
   ```bash
   docker-compose up -d
   ```

4. Set up NGINX and SSL:
   ```bash
   # Install NGINX
   sudo apt install nginx

   # Install Certbot
   sudo apt install certbot python3-certbot-nginx

   # Configure NGINX and obtain certificates
   sudo certbot --nginx
   ```

5. Configure DNS records in your domain provider pointing to your server IP

## Contributing

Contributions are welcome! To contribute to this project:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please ensure your code adheres to the project's style guide and includes appropriate tests.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Original e-commerce application code from [GreatStack](https://greatstack.dev)
- Docker and container orchestration inspired by best practices from the Docker community
- AWS deployment strategies from AWS Well-Architected Framework
- NGINX configuration patterns from NGINX documentation

---

**Contact Information:**
- **Name**: Aimable M.
- **Portfolio**: [aimablem.dev](https://aimablem.dev)
- **GitHub**: [github.com/aimablM](https://github.com/aimablM)
- **Email**: aimable.mugwaneza@gmail.com
