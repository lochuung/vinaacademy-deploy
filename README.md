# VinaAcademy Deployment

This repository contains Docker configuration for deploying the VinaAcademy project, which consists of a Spring Boot backend and a Next.js frontend.

## Project Structure

The project is configured with the following services:

- **PostgreSQL**: Database server for storing application data
- **Redis**: In-memory data structure store used for caching
- **Backend**: Spring Boot application
- **Frontend**: Next.js application

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)
- Git
- Nginx for production deployment
- Domain name with access to DNS settings

## Setup Instructions

### 1. Clone the Repositories

```bash
# Clone this deployment repository
git clone https://github.com/lochuung/vinaacademy-deploy.git vinaacademy-deploy
cd vinaacademy-deploy

# Clone the backend and frontend repositories from the 'deploy' branch
git clone -b deploy https://github.com/lochuung/vinaacademy-backend.git backend
git clone -b deploy https://github.com/lochuung/vinaacademy-frontend.git frontend
```

### 2. Environment Configuration

Create a `.env` file in the root directory with the following variables:

```
# Database configuration
POSTGRES_DB=vinaacademy
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password

# Application secrets
HMAC_SECRET=your_hmac_secret_key

# File storage paths
UPLOAD_DIR=/vinaacademy/uploads
HLS_OUTPUT_DIR=/vinaacademy/hls

# Frontend configuration
FRONTEND_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1
NEXT_PUBLIC_SITE_URL=http://localhost:3000

# Email configuration (for sending emails)
GMAIL_USERNAME=your_gmail_username@gmail.com
GMAIL_PASSWORD=your_app_password
```

### 3. Build and Start the Application

```bash
docker-compose up -d
```

This will build and start all services defined in the docker-compose.yml file.

### 4. Access the Application (Development)

- Frontend: http://localhost:3000
- Backend API: http://localhost:8080/api/v1

## Production Deployment with Nginx and SSL

### 1. Configure Cloudflare DNS

1. Create a Cloudflare account and add your domain
2. In your Cloudflare DNS settings, add an A record:
   - Type: A
   - Name: vinaacademy (or subdomain of your choice)
   - Content: Your server's public IP address
   - Proxy status: DNS only (gray cloud) initially
3. Set the TTL to Auto or 1 hour

### 2. Configure Nginx

1. Create an Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/vinaacademy
```

2. Add the following configuration:

```nginx
server {
    listen 80;
    server_name vinaacademy.huuloc.id.vn;
    
    client_max_body_size 2048M;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

3. Create a symbolic link to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/vinaacademy /etc/nginx/sites-enabled/
```

4. Test the Nginx configuration:

```bash
sudo nginx -t
```

5. If the test is successful, restart Nginx:

```bash
sudo systemctl restart nginx
```

### 3. Configure SSL with Let's Encrypt

1. Install Certbot:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

2. Obtain and configure SSL certificate:

```bash
sudo certbot --nginx -d vinaacademy.huuloc.id.vn
```

3. Follow the prompts to complete the process and choose to redirect HTTP traffic to HTTPS

4. Certbot will automatically update your Nginx configuration with SSL settings

5. After successful SSL configuration, go back to Cloudflare and enable proxy (orange cloud) for additional protection

### 4. Update Environment Variables for Production

Update your `.env` file with production values:

```
# Frontend configuration
FRONTEND_URL=https://vinaacademy.huuloc.id.vn
NEXT_PUBLIC_API_URL=https://vinaacademy.huuloc.id.vn/api/v1
NEXT_PUBLIC_SITE_URL=https://vinaacademy.huuloc.id.vn
```

5. Rebuild and restart your services:

```bash
docker-compose up -d --build
```

## Volume Information

The deployment configuration creates the following Docker volumes:

- `postgres_data`: Persists PostgreSQL database data
- `redis_data`: Persists Redis data
- `uploads_data`: Stores uploaded files
- `hls_data`: Stores HLS video streaming files

## Service Configuration

### Backend Service

The backend service uses a Spring Boot application with the following configuration:

- Database connection to PostgreSQL
- Redis for caching
- File storage for uploads and HLS video streaming
- Email service configuration

### Frontend Service

The frontend service uses a Next.js application with:

- API proxy configuration to forward requests to the backend
- Environment variables for API URL and site URL

## Troubleshooting

### Database Connection Issues

If the backend cannot connect to the database, check:

```bash
# View logs from the backend service
docker-compose logs backend

# Check if PostgreSQL is running
docker-compose ps postgres
```

### SSL Certificate Issues

If you encounter issues with SSL certificates:

```bash
# Check Certbot certificate status
sudo certbot certificates

# Renew certificates (can be done early for testing)
sudo certbot renew --dry-run
```

### Email Configuration

To use Gmail for sending emails, you need to:

1. Enable 2-Step Verification for your Google account
2. Create an App Password: https://myaccount.google.com/apppasswords
3. Use the generated App Password in your `.env` file

## Maintenance

### Updating Services

To update the services with the latest code:

```bash
# Pull latest code for backend and frontend from the deploy branch
cd backend && git pull origin deploy && cd ..
cd frontend && git pull origin deploy && cd ..

# Rebuild and restart the services
docker-compose up -d --build
```

### Certificate Renewal

Let's Encrypt certificates expire after 90 days. Certbot installs a timer that will automatically renew your certificates before they expire. You can test the renewal process with:

```bash
sudo certbot renew --dry-run
```

### Backup Database

To backup the PostgreSQL database:

```bash
docker-compose exec postgres pg_dump -U postgres vinaacademy > backup.sql
```

## License

[Your License Information]
