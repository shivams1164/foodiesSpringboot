# Render Deployment Guide

This guide explains how to deploy the Foodies API to Render.com.

## Prerequisites

- GitHub repository with this code
- Render.com account
- Docker image built and ready

## Deployment Steps

### 1. Connect GitHub Repository to Render

1. Go to [Render Dashboard](https://dashboard.render.com)
2. Click "New +" → "Web Service"
3. Connect your GitHub repository
4. Select this repository

### 2. Configure Service

- **Service Name**: `foodies-api` (or your preferred name)
- **Region**: Choose closest to your users
- **Branch**: `main` (or your default branch)
- **Runtime**: `Docker`
- **Build Command**: (leave empty - uses Dockerfile)
- **Start Command**: (leave empty - uses Dockerfile ENTRYPOINT)

### 3. Set Environment Variables

In the "Environment" section, add the following variables:

```
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/foodiesdb?retryWrites=true&w=majority
JWT_SECRET=your_jwt_secret_key_here
AWS_ACCESS_KEY=your_aws_access_key
AWS_SECRET_KEY=your_aws_secret_key
AWS_REGION=us-east-1
AWS_S3_BUCKET=your_s3_bucket_name
RAZORPAY_KEY=your_razorpay_key
RAZORPAY_SECRET=your_razorpay_secret
```

**IMPORTANT**: Replace all values with actual credentials from your `.env` file.

### 4. Configure Port

Render automatically detects port 8080 from the Dockerfile EXPOSE instruction.

### 5. Deploy

Click "Create Web Service" to start deployment. Render will:
1. Build Docker image from Dockerfile
2. Push to Render's registry
3. Deploy container
4. Start health checks

### 6. Monitor Deployment

- Check logs in Render dashboard
- Verify health checks passing
- Test endpoints once "Live" status appears

## Health Check

The Dockerfile includes a health check that:
- Tests `http://localhost:8080/actuator/health`
- Runs every 30 seconds
- Requires 3 consecutive successes to mark healthy
- Waits 40 seconds before first check

**Note**: If health checks fail, ensure Spring Boot Actuator is enabled in your application.

## Local Testing with Docker

To test the Docker image locally before deploying:

```bash
# Build image
docker build -t foodies-api .

# Run container with environment variables
docker run -p 8080:8080 \
  -e MONGODB_URI="mongodb+srv://..." \
  -e JWT_SECRET="your_secret" \
  -e AWS_ACCESS_KEY="your_key" \
  -e AWS_SECRET_KEY="your_secret" \
  -e AWS_REGION="us-east-1" \
  -e AWS_S3_BUCKET="bucket_name" \
  -e RAZORPAY_KEY="key" \
  -e RAZORPAY_SECRET="secret" \
  foodies-api

# Test health endpoint
curl http://localhost:8080/actuator/health
```

## Troubleshooting

### Container won't start
- Check logs: `docker logs <container_id>`
- Verify all required environment variables are set
- Ensure MongoDB Atlas connection string is correct

### Health checks failing
- Verify application started successfully (check logs)
- Ensure actuator health endpoint is accessible
- Check network connectivity to MongoDB

### Slow startup
- First deployment may take 3-5 minutes
- Subsequent deployments usually faster (cached layers)

## Build Details

The Dockerfile uses a **multi-stage build**:
- **Stage 1 (Builder)**: Builds JAR using Maven wrapper
- **Stage 2 (Runtime)**: Minimal image with only JRE and JAR
- **Benefit**: Smaller final image (~500MB vs 1.5GB+)

## Port Configuration

The application runs on port 8080 (default Spring Boot).
Render automatically assigns a public URL and routes traffic to this port.

## Database Connection

MongoDB Atlas connection string format:
```
mongodb+srv://username:password@cluster-name.mongodb.net/database?retryWrites=true&w=majority
```

Ensure your MongoDB Atlas network access allows Render's IP ranges:
- Go to MongoDB Atlas → Network Access
- Add Render's IP or allow "0.0.0.0/0" (less secure)

## Auto-Deployment

Render supports auto-deployment on git push:
- Navigate to service settings
- Enable "Auto-Deploy" 
- Whenever you push to the connected branch, Render rebuilds and deploys

## Rollback

If deployment fails:
1. Go to service → "Deploys"
2. Click "Redeploy" on a previous successful deploy

---

**For more help**: See [Render Docker Documentation](https://render.com/docs/docker)
