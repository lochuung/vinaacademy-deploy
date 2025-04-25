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

## Setup Instructions

### 1. Clone the Repositories

```bash
# Clone this deployment repository
git clone https://github.com/lochuung/vinaacademy-deploy.git vinaacademy-deploy
cd vinaacademy-deploy

# Clone the backend and frontend repositories
git clone https://github.com/lochuung/vinaacademy-backend.git backend
git clone https://github.com/lochuung/vinaacademy-frontend.git frontend
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

### 4. Access the Application

- Frontend: http://localhost:3000
- Backend API: http://localhost:8080/api/v1

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

### Email Configuration

To use Gmail for sending emails, you need to:

1. Enable 2-Step Verification for your Google account
2. Create an App Password: https://myaccount.google.com/apppasswords
3. Use the generated App Password in your `.env` file

## Maintenance

### Updating Services

To update the services with the latest code:

```bash
# Pull latest code for backend and frontend
cd backend && git pull && cd ..
cd frontend && git pull && cd ..

# Rebuild and restart the services
docker-compose up -d --build
```

### Backup Database

To backup the PostgreSQL database:

```bash
docker-compose exec postgres pg_dump -U postgres vinaacademy > backup.sql
```

## License

[Your License Information]
