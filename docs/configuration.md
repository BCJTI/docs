# Configuration Guide

This guide covers all configuration options available in SmartComex.

## Overview

SmartComex offers extensive configuration options to customize the platform to your business needs. Configuration can be done through:
- Web-based admin interface
- Configuration files (for on-premises installations)
- Environment variables
- API endpoints

## Basic Configuration

### Business Information
Configure your basic business details:

- **Company Name**: Your business name as it appears to customers
- **Business Address**: Physical address for legal and shipping purposes
- **Contact Information**: Phone, email, and support contact details
- **Tax Information**: Tax ID, VAT numbers, and tax calculation settings
- **Timezone**: Default timezone for your business operations

### Store Settings
- **Store Name**: Display name for your online store
- **Store URL**: Primary domain for your store
- **Default Language**: Primary language for customer-facing content
- **Default Currency**: Base currency for pricing and transactions
- **Logo and Branding**: Upload and configure your brand assets

## Payment Configuration

### Payment Gateways
SmartComex supports multiple payment processors:

#### Stripe Configuration
```json
{
  "stripe": {
    "publishable_key": "pk_live_...",
    "secret_key": "sk_live_...",
    "webhook_secret": "whsec_...",
    "enabled": true
  }
}
```

#### PayPal Configuration
```json
{
  "paypal": {
    "client_id": "your_client_id",
    "client_secret": "your_client_secret",
    "mode": "live",
    "enabled": true
  }
}
```

#### Square Configuration
```json
{
  "square": {
    "application_id": "your_app_id",
    "access_token": "your_access_token",
    "location_id": "your_location_id",
    "enabled": true
  }
}
```

### Payment Settings
- **Accepted Payment Methods**: Credit cards, digital wallets, bank transfers
- **Payment Processing**: Real-time vs. batch processing
- **Refund Policies**: Automatic refund rules and manual approval workflows
- **Currency Settings**: Multi-currency support and exchange rates

## Shipping Configuration

### Shipping Providers
Configure shipping integrations:

#### FedEx
```json
{
  "fedex": {
    "key": "your_fedex_key",
    "password": "your_password",
    "account_number": "your_account",
    "meter_number": "your_meter",
    "enabled": true
  }
}
```

#### UPS
```json
{
  "ups": {
    "username": "your_username",
    "password": "your_password",
    "access_license_number": "your_license",
    "enabled": true
  }
}
```

### Shipping Rules
- **Shipping Zones**: Define geographical shipping areas
- **Shipping Methods**: Standard, expedited, overnight options
- **Shipping Rates**: Flat rate, weight-based, or calculated rates
- **Free Shipping**: Minimum order thresholds and promotional rules

## Inventory Configuration

### Stock Management
- **Inventory Tracking**: Enable/disable inventory tracking per product
- **Low Stock Alerts**: Set threshold levels for automated alerts
- **Backorder Handling**: Allow/disallow backorders and notification settings
- **Multi-location Inventory**: Configure warehouse and store locations

### Product Settings
- **SKU Format**: Define SKU generation patterns
- **Product Variants**: Configure size, color, and other variant options
- **Product Categories**: Set up hierarchical category structures
- **Product Attributes**: Custom fields and specifications

## Customer Configuration

### Customer Accounts
- **Registration Requirements**: Mandatory vs. optional registration
- **Account Verification**: Email verification and approval workflows
- **Customer Groups**: VIP, wholesale, retail customer segmentation
- **Loyalty Programs**: Points, tiers, and reward configurations

### Customer Communication
- **Email Templates**: Customize transactional email templates
- **Notification Settings**: Configure automated notifications
- **Customer Support**: Help desk integration and chat settings
- **Marketing Preferences**: Newsletter and promotional email settings

## Security Configuration

### Authentication
- **Password Policies**: Minimum requirements and complexity rules
- **Two-Factor Authentication**: Enable 2FA for admin and customer accounts
- **Session Management**: Timeout settings and concurrent session limits
- **API Security**: Rate limiting and authentication methods

### Data Protection
- **SSL/TLS Configuration**: Certificate management and encryption settings
- **Data Backup**: Automated backup schedules and retention policies
- **GDPR Compliance**: Privacy settings and data processing consent
- **PCI Compliance**: Payment data security configurations

## Advanced Configuration

### API Settings
- **API Rate Limiting**: Configure request limits per endpoint
- **Webhook Configuration**: Set up event notifications
- **API Keys**: Generate and manage API access keys
- **CORS Settings**: Cross-origin resource sharing policies

### Performance Settings
- **Caching**: Redis or Memcached configuration
- **CDN Integration**: Content delivery network settings
- **Database Optimization**: Query caching and indexing
- **Image Optimization**: Compression and resizing settings

### Integration Settings
- **Analytics**: Google Analytics, Facebook Pixel integration
- **Marketing Tools**: Email marketing platform connections
- **Accounting Software**: QuickBooks, Xero synchronization
- **CRM Integration**: Salesforce, HubSpot connections

## Environment-Specific Configuration

### Development Environment
```env
NODE_ENV=development
DEBUG=true
LOG_LEVEL=debug
DATABASE_URL=postgresql://localhost:5432/smartcomex_dev
REDIS_URL=redis://localhost:6379
```

### Production Environment
```env
NODE_ENV=production
DEBUG=false
LOG_LEVEL=info
DATABASE_URL=postgresql://prod-db:5432/smartcomex
REDIS_URL=redis://prod-redis:6379
SSL_ENABLED=true
```

## Configuration Management

### Configuration Files
For on-premises installations, configuration is managed through:
- `config/default.json`: Default configuration
- `config/production.json`: Production overrides
- `config/development.json`: Development overrides
- `.env`: Environment-specific variables

### Web Interface
Most settings can be configured through the admin web interface:
1. Navigate to Settings → Configuration
2. Select the configuration category
3. Update settings as needed
4. Save and apply changes

### API Configuration
Use the Configuration API for programmatic updates:
```bash
curl -X PUT /api/config/payment \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{"stripe": {"enabled": true}}'
```

## Validation and Testing

### Configuration Validation
- **Syntax Checking**: Validate JSON/YAML configuration syntax
- **Dependency Checking**: Verify required services and integrations
- **Security Scanning**: Check for security misconfigurations
- **Performance Impact**: Assess configuration impact on performance

### Testing Configuration Changes
1. **Staging Environment**: Test changes in a staging environment first
2. **Gradual Rollout**: Apply changes incrementally
3. **Monitoring**: Monitor system performance after changes
4. **Rollback Plan**: Have a rollback strategy ready

## Best Practices

### Security Best Practices
- Store sensitive configuration in environment variables
- Use strong passwords and rotate API keys regularly
- Enable audit logging for configuration changes
- Implement proper access controls for configuration management

### Performance Best Practices
- Use appropriate caching strategies
- Configure database connection pooling
- Optimize image and asset delivery
- Monitor and tune based on usage patterns

## Support

For configuration assistance:
- Review the [FAQ](support/faq.md) for common configuration questions
- Check the [Troubleshooting Guide](support/troubleshooting.md)
- Contact [Technical Support](support/contact.md) for complex configuration issues