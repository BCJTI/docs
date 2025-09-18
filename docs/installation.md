# Installation Guide

This guide covers the installation and initial setup of SmartComex.

## System Requirements

### Minimum Requirements
- Operating System: Windows 10+, macOS 10.15+, or Linux (Ubuntu 18.04+)
- RAM: 4GB minimum, 8GB recommended
- Storage: 10GB free space
- Internet connection: Broadband recommended

### Software Dependencies
- Node.js 16.x or higher
- npm 8.x or higher
- Modern web browser (Chrome, Firefox, Safari, Edge)

## Installation Methods

### Cloud-Based Installation (Recommended)

SmartComex is primarily offered as a cloud-based solution:

1. **Sign Up**: Visit [SmartComex.com](https://smartcomex.com) and create an account
2. **Choose Plan**: Select the plan that fits your business needs
3. **Setup**: Follow the guided setup process
4. **Verification**: Verify your email and complete account setup

### On-Premises Installation

For enterprise customers requiring on-premises deployment:

#### Prerequisites
- Docker 20.x or higher
- Docker Compose 2.x or higher
- SSL certificate for your domain

#### Installation Steps

1. **Download the Installation Package**
   ```bash
   wget https://releases.smartcomex.com/enterprise/latest/smartcomex-enterprise.tar.gz
   tar -xzf smartcomex-enterprise.tar.gz
   cd smartcomex-enterprise
   ```

2. **Configure Environment**
   ```bash
   cp .env.example .env
   nano .env  # Edit configuration as needed
   ```

3. **Start Services**
   ```bash
   docker-compose up -d
   ```

4. **Initialize Database**
   ```bash
   docker-compose exec app npm run migrate
   docker-compose exec app npm run seed
   ```

5. **Access the Application**
   Open your browser and navigate to `https://your-domain.com`

## Post-Installation Setup

### 1. Initial Configuration
- Complete the setup wizard
- Configure your business information
- Set up administrator accounts

### 2. Security Setup
- Enable two-factor authentication
- Configure SSL certificates
- Set up backup procedures

### 3. Integration Setup
- Configure payment gateways
- Set up shipping providers
- Connect third-party services

## Verification

To verify your installation:

1. **Health Check**
   ```bash
   curl https://your-domain.com/health
   ```

2. **Admin Access**
   - Log in to the admin panel
   - Verify all services are running
   - Check system status dashboard

## Troubleshooting

Common installation issues:

### Port Conflicts
If you encounter port conflicts, modify the ports in your `.env` file:
```
HTTP_PORT=8080
HTTPS_PORT=8443
```

### Database Connection Issues
Verify database credentials and connectivity:
```bash
docker-compose logs db
```

### SSL Certificate Issues
Ensure your SSL certificates are properly configured and not expired.

## Next Steps

After successful installation:
- Complete the [Getting Started Guide](getting-started.md)
- Review [Configuration Options](configuration.md)
- Set up [User Accounts](user-guide/admin-guide.md)

## Support

For installation support:
- Check our [FAQ](support/faq.md)
- Contact [Technical Support](support/contact.md)
- Review [Troubleshooting Guide](support/troubleshooting.md)