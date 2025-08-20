# SmartCoach Setup & Deployment Guide

## Overview
This guide provides step-by-step instructions for setting up and deploying the SmartCoach financial coaching application.

## Prerequisites

### System Requirements
- **Node.js**: Version 18.0.0 or higher
- **PostgreSQL**: Version 12.0 or higher
- **npm**: Version 8.0.0 or higher
- **Git**: For version control

### Operating Systems
- Windows 10/11
- macOS 10.15 or higher
- Ubuntu 20.04 LTS or higher

## Local Development Setup

### 1. Clone the Repository
```bash
git clone <repository-url>
cd smartcoach
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Environment Configuration
Create a `.env.local` file in the root directory:

```bash
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/smartcoach"

# Authentication (Clerk)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="pk_test_..."
CLERK_SECRET_KEY="sk_test_..."

# OpenAI API
OPENAI_API_KEY="sk-..."

# Email (Resend)
RESEND_API_KEY="re_..."

# Telegram Bot (Optional)
TELEGRAM_BOT_TOKEN=""

# App Configuration
APP_URL="http://localhost:3000"

# Encryption
ENCRYPTION_KEY="your-32-character-encryption-key-here"
```

### 4. Database Setup

#### Install PostgreSQL
**Windows:**
- Download from [PostgreSQL Downloads](https://www.postgresql.org/download/windows/)
- Use the installer and follow the setup wizard
- Remember the password you set for the `postgres` user

**macOS:**
```bash
brew install postgresql
brew services start postgresql
```

**Ubuntu:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

#### Create Database
```bash
# Connect to PostgreSQL
sudo -u postgres psql

# Create database and user
CREATE DATABASE smartcoach;
CREATE USER smartcoach_user WITH ENCRYPTED PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE smartcoach TO smartcoach_user;
ALTER USER smartcoach_user CREATEDB;
\q
```

#### Update Environment Variables
Update your `.env.local` with the correct database credentials:
```bash
DATABASE_URL="postgresql://smartcoach_user:your_password@localhost:5432/smartcoach"
```

### 5. Database Migration
```bash
# Generate Prisma client
npm run db:generate

# Run database migrations
npm run db:migrate

# Seed database with sample data
npm run db:seed
```

### 6. Start Development Server
```bash
npm run dev
```

The application will be available at `http://localhost:3000`

## Production Deployment

### Option 1: Vercel (Recommended)

#### 1. Prepare for Deployment
```bash
# Build the application
npm run build

# Test the build locally
npm start
```

#### 2. Deploy to Vercel
1. Install Vercel CLI: `npm i -g vercel`
2. Login to Vercel: `vercel login`
3. Deploy: `vercel --prod`

#### 3. Environment Variables in Vercel
Set the following environment variables in your Vercel project dashboard:
- `DATABASE_URL`
- `CLERK_SECRET_KEY`
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
- `OPENAI_API_KEY`
- `RESEND_API_KEY`
- `ENCRYPTION_KEY`

### Option 2: Docker Deployment

#### 1. Create Dockerfile
```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json package-lock.json* ./
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
ENV NEXT_TELEMETRY_DISABLED 1

RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

#### 2. Create Docker Compose
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://smartcoach_user:password@db:5432/smartcoach
      - NODE_ENV=production
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=smartcoach
      - POSTGRES_USER=smartcoach_user
      - POSTGRES_PASSWORD=your_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

volumes:
  postgres_data:
```

#### 3. Deploy with Docker
```bash
# Build and start services
docker-compose up -d

# Run database migrations
docker-compose exec app npm run db:migrate

# Seed database
docker-compose exec app npm run db:seed
```

### Option 3: Traditional Server Deployment

#### 1. Server Requirements
- Ubuntu 20.04 LTS or higher
- 2GB RAM minimum (4GB recommended)
- 20GB storage
- Domain name with SSL certificate

#### 2. Server Setup
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Install Nginx
sudo apt install nginx

# Install PM2 for process management
sudo npm install -g pm2
```

#### 3. Application Deployment
```bash
# Clone repository
git clone <repository-url> /var/www/smartcoach
cd /var/www/smartcoach

# Install dependencies
npm install

# Build application
npm run build

# Set up environment variables
cp .env.example .env
# Edit .env with production values

# Set up database
npm run db:migrate
npm run db:seed

# Start with PM2
pm2 start npm --name "smartcoach" -- start
pm2 startup
pm2 save
```

#### 4. Nginx Configuration
```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Environment Variables Reference

### Required Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@host:5432/db` |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk public key | `pk_test_...` |
| `CLERK_SECRET_KEY` | Clerk secret key | `sk_test_...` |
| `OPENAI_API_KEY` | OpenAI API key | `sk-...` |
| `ENCRYPTION_KEY` | 32-character encryption key | `your-32-char-key-here` |

### Optional Variables
| Variable | Description | Default |
|----------|-------------|---------|
| `RESEND_API_KEY` | Resend email service key | - |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token | - |
| `APP_URL` | Application URL | `http://localhost:3000` |
| `NODE_ENV` | Environment mode | `development` |

## Security Considerations

### 1. Environment Variables
- Never commit `.env` files to version control
- Use strong, unique passwords for database
- Rotate API keys regularly
- Use environment-specific configurations

### 2. Database Security
- Use strong passwords
- Limit database access to application only
- Enable SSL connections in production
- Regular backups

### 3. Application Security
- Keep dependencies updated
- Enable HTTPS in production
- Implement rate limiting
- Regular security audits

## Monitoring & Maintenance

### 1. Health Checks
```bash
# Check application status
curl http://localhost:3000/api/health

# Check database connection
npm run db:studio
```

### 2. Logs
```bash
# Application logs
pm2 logs smartcoach

# Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Database logs
sudo tail -f /var/log/postgresql/postgresql-*.log
```

### 3. Backups
```bash
# Database backup
pg_dump -U smartcoach_user -h localhost smartcoach > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore database
psql -U smartcoach_user -h localhost smartcoach < backup_file.sql
```

### 4. Updates
```bash
# Pull latest changes
git pull origin main

# Install dependencies
npm install

# Run migrations
npm run db:migrate

# Build and restart
npm run build
pm2 restart smartcoach
```

## Troubleshooting

### Common Issues

#### 1. Database Connection Failed
- Verify PostgreSQL is running
- Check connection string in `.env`
- Ensure database and user exist
- Check firewall settings

#### 2. Build Errors
- Clear node_modules: `rm -rf node_modules && npm install`
- Check Node.js version compatibility
- Verify all dependencies are installed

#### 3. Authentication Issues
- Verify Clerk keys are correct
- Check environment variables
- Ensure Clerk app is properly configured

#### 4. File Upload Issues
- Check file size limits
- Verify file format support
- Check storage permissions

### Getting Help
1. Check the application logs
2. Review the test plan (`test-app.md`)
3. Verify environment configuration
4. Check database connectivity
5. Review browser console for errors

## Support & Resources

- **Documentation**: Check the README.md for detailed feature descriptions
- **Issues**: Report bugs and feature requests through the issue tracker
- **Community**: Join discussions in the project community
- **Security**: Report security vulnerabilities privately

## Conclusion

This setup guide covers the essential steps to get SmartCoach running in both development and production environments. Follow the security best practices and maintain regular backups to ensure a stable and secure application.

For additional help or questions, refer to the project documentation or reach out to the development team.
