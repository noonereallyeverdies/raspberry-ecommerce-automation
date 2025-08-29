# Development Setup Guide - Mac to Pi Deployment

## Stage 1: Local Development (Your Mac)

### 1.1 Project Structure
```
raspberry-automation/
├── docker/
│   ├── docker-compose.yml
│   ├── docker-compose.dev.yml
│   └── docker-compose.prod.yml
├── appwrite/
│   ├── functions/
│   ├── collections/
│   └── init-script.sh
├── n8n/
│   ├── workflows/
│   └── credentials/
├── services/
│   ├── rate-limiter/
│   ├── webhook-handler/
│   └── monitoring/
├── scripts/
│   ├── deploy.sh
│   ├── backup.sh
│   └── health-check.sh
├── config/
│   ├── .env.example
│   ├── .env.development
│   └── .env.production
└── docs/
```

### 1.2 Mac Development Setup
```bash
# Install required tools on Mac
brew install docker docker-compose node git

# Clone your repository
git clone https://github.com/yourusername/raspberry-automation
cd raspberry-automation

# Copy environment template
cp config/.env.example config/.env.development

# Start development stack (x86 versions)
docker-compose -f docker/docker-compose.dev.yml up -d
```

### 1.3 Development Docker Compose (Mac)
```yaml
# docker/docker-compose.dev.yml
version: '3.8'

services:
  # Use x86 images for Mac development
  appwrite:
    image: appwrite/appwrite:1.5-amd64
    container_name: appwrite_dev
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - appwrite-uploads:/storage/uploads
      - appwrite-cache:/storage/cache
      - appwrite-config:/storage/config
      - appwrite-certificates:/storage/certificates
      - appwrite-functions:/storage/functions
    environment:
      - _APP_ENV=development
      - _APP_OPENSSL_KEY_V1=your-32-char-key
      - _APP_DOMAIN=localhost
      - _APP_DOMAIN_TARGET=localhost
    
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_dev
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=admin
    volumes:
      - ./n8n:/home/node/.n8n
      
  redis:
    image: redis:7-alpine
    container_name: redis_dev
    ports:
      - "6379:6379"

volumes:
  appwrite-uploads:
  appwrite-cache:
  appwrite-config:
  appwrite-certificates:
  appwrite-functions:
```

---

## Stage 2: Testing & Configuration

### 2.1 Develop Locally with Pi Compatibility in Mind
```javascript
// Always use environment variables
const config = {
  appwriteEndpoint: process.env.APPWRITE_ENDPOINT || 'http://localhost/v1',
  appwriteProject: process.env.APPWRITE_PROJECT,
  // Use relative paths that work on both systems
  dataPath: process.env.DATA_PATH || './data',
  // Account for Pi's slower performance
  timeout: process.env.NODE_ENV === 'production' ? 30000 : 5000
};
```

### 2.2 Test ARM Compatibility
```bash
# Build multi-architecture images locally
docker buildx create --name multiarch --use
docker buildx build --platform linux/arm64 -t myapp:arm64 .

# Test with QEMU emulation (slower but works)
docker run --platform linux/arm64 myapp:arm64
```

### 2.3 Create n8n Workflows Locally
- Develop all workflows in local n8n instance
- Export as JSON files to `n8n/workflows/`
- Version control all workflow files
- Test with mock data first

---

## Stage 3: Pi Deployment

### 3.1 Prepare the Pi (One-Time Setup)
```bash
# On your Mac, prepare the SSD
# Download Ubuntu Server 22.04 ARM64
wget https://ubuntu.com/download/raspberry-pi/ubuntu-22.04-arm64.img.xz

# Flash to SSD (use Raspberry Pi Imager or dd)
# Enable SSH by adding empty 'ssh' file to boot partition
touch /Volumes/boot/ssh

# Configure WiFi (optional, prefer Ethernet)
cat > /Volumes/boot/network-config << EOF
version: 2
ethernets:
  eth0:
    dhcp4: true
    optional: true
wifis:
  wlan0:
    dhcp4: true
    optional: true
    access-points:
      "YOUR_WIFI_SSID":
        password: "YOUR_WIFI_PASSWORD"
EOF
```

### 3.2 Initial Pi Configuration
```bash
# Boot Pi with SSD, SSH in
ssh ubuntu@raspberrypi.local
# Default password: ubuntu (will force change)

# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
logout
# Log back in

# Install essential tools
sudo apt install -y git vim htop ncdu
```

### 3.3 Deploy Your Project
```bash
# On Pi: Clone your repository
git clone https://github.com/yourusername/raspberry-automation
cd raspberry-automation

# Copy production environment
cp config/.env.example config/.env.production
nano config/.env.production  # Edit with your values

# Run deployment script
chmod +x scripts/deploy.sh
./scripts/deploy.sh
```

### 3.4 Deployment Script
```bash
#!/bin/bash
# scripts/deploy.sh

set -e

echo "🚀 Starting Raspberry Pi Deployment"

# Check if running on ARM64
if [ "$(uname -m)" != "aarch64" ]; then
    echo "❌ Error: This script must run on ARM64 architecture"
    exit 1
fi

# Load environment
source config/.env.production

# Pull latest changes
echo "📦 Pulling latest code..."
git pull origin main

# Install Appwrite if not exists
if [ ! -d "./appwrite" ]; then
    echo "📦 Installing Appwrite..."
    docker run -it --rm \
        --volume /var/run/docker.sock:/var/run/docker.sock \
        --volume "$(pwd)"/appwrite:/usr/src/code/appwrite:rw \
        --entrypoint="install" \
        appwrite/appwrite:latest
fi

# Start services
echo "🔧 Starting services..."
docker-compose -f docker/docker-compose.prod.yml up -d

# Wait for services
echo "⏳ Waiting for services to be ready..."
sleep 30

# Run health checks
./scripts/health-check.sh

# Import n8n workflows
echo "📝 Importing n8n workflows..."
for workflow in n8n/workflows/*.json; do
    echo "Importing $(basename $workflow)..."
    # Import logic here
done

echo "✅ Deployment complete!"
echo "🌐 Access points:"
echo "  - Appwrite: http://$(hostname -I | awk '{print $1}')"
echo "  - n8n: http://$(hostname -I | awk '{print $1}'):5678"
```

---

## Development Workflow

### Daily Development Process

```mermaid
graph LR
    A[Write Code on Mac] --> B[Test Locally]
    B --> C[Commit to Git]
    C --> D[Push to GitHub]
    D --> E[SSH to Pi]
    E --> F[Pull & Deploy]
    F --> G[Monitor & Test]
```

### Best Practices

1. **Never edit directly on Pi**
   - Always use Git for deployment
   - Keep Pi as read-only as possible

2. **Use Docker volumes for data**
   ```yaml
   volumes:
     - ./data:/data  # Development
     - /mnt/ssd/data:/data  # Production on Pi
   ```

3. **Separate configs**
   ```bash
   .env.development  # Mac local testing
   .env.production   # Pi production
   ```

4. **Version control everything**
   ```
   - All Docker configs
   - All n8n workflows
   - All Appwrite functions
   - All deployment scripts
   ```

---

## Backup Strategy During Development

### Before Major Changes
```bash
# On Pi: Create snapshot before updates
sudo ./scripts/backup.sh full

# Backup script
#!/bin/bash
# scripts/backup.sh

BACKUP_DIR="/mnt/ssd/backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# Backup Docker volumes
docker run --rm \
  -v appwrite_data:/data \
  -v $BACKUP_DIR:/backup \
  alpine tar czf /backup/appwrite.tar.gz /data

# Backup configurations
tar czf $BACKUP_DIR/config.tar.gz config/
tar czf $BACKUP_DIR/n8n.tar.gz n8n/

echo "Backup saved to $BACKUP_DIR"
```

---

## Troubleshooting Connection

### Can't Connect to Pi?
```bash
# On Mac: Find Pi's IP
arp -a | grep raspberry
# or
dns-sd -B _ssh._tcp

# Test connection
ping raspberrypi.local
ssh ubuntu@raspberrypi.local

# If SSH fails, check:
# 1. Is Pi powered on?
# 2. Is Ethernet/WiFi connected?
# 3. Is SSH enabled?
# 4. Correct username/password?
```

### Remote Development Options

#### Option 1: VS Code Remote SSH (Recommended)
```bash
# Install Remote-SSH extension in VS Code
# Add SSH config
Host raspberry
    HostName raspberrypi.local
    User ubuntu
    Port 22

# Connect via VS Code and edit remotely
# BUT only for quick fixes, not main development
```

#### Option 2: Continuous Deployment
```bash
# Use GitHub Actions for auto-deploy
# .github/workflows/deploy.yml
name: Deploy to Pi
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Pi
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PI_HOST }}
          username: ${{ secrets.PI_USER }}
          key: ${{ secrets.PI_SSH_KEY }}
          script: |
            cd /home/ubuntu/raspberry-automation
            git pull
            docker-compose down
            docker-compose up -d
```

---

## Summary: Why This Approach Works

### ✅ Advantages of Local Dev → Pi Deploy:

1. **10x faster development** on your Mac
2. **Version controlled** - never lose work
3. **Easy rollback** if something breaks
4. **Test safely** without breaking Pi
5. **Professional workflow** used in real companies
6. **Collaboration ready** if you add team members

### ❌ Why Direct Pi Development Fails:

1. **Corrupted SD cards** from excessive writes
2. **Slow development** (compile times, Docker builds)
3. **No rollback** when things break
4. **Limited debugging** tools
5. **Risk of breaking** production system

### 🎯 The Golden Rule:
**Develop on Mac → Test Locally → Deploy to Pi → Never Edit on Pi**

This approach has saved countless hours and prevented many disasters in production Raspberry Pi deployments!