# Authentication

The SmartComex API uses API key authentication to secure access to your data. This guide covers how to authenticate your requests and manage your API keys securely.

## API Key Authentication

### Overview
All API requests must include an authentication header with a valid API key. API keys are unique to your account and provide access to your store's data and functionality.

### Authentication Header
Include your API key in the `Authorization` header using the Bearer token format:

```http
Authorization: Bearer your_api_key_here
```

### Example Request
```bash
curl -H "Authorization: Bearer sk_live_1234567890abcdef" \
     https://api.smartcomex.com/v1/products
```

## Managing API Keys

### Creating API Keys
1. Log in to your SmartComex dashboard
2. Navigate to Settings → API Keys
3. Click "Create New API Key"
4. Enter a descriptive name for the key
5. Select the appropriate permissions
6. Click "Generate Key"

### API Key Types
- **Public Keys** (`pk_`): Safe to use in client-side code, limited permissions
- **Secret Keys** (`sk_`): Full access, must be kept secure on your server
- **Restricted Keys** (`rk_`): Limited permissions, recommended for specific integrations

### Key Permissions
Control what each API key can access:
- **Read**: View data (products, orders, customers)
- **Write**: Create and update data
- **Delete**: Remove data (use with caution)
- **Admin**: Full administrative access

### Environment Keys
Use different keys for different environments:
- **Live/Production** (`_live_`): For production environment
- **Test/Sandbox** (`_test_`): For development and testing

## Security Best Practices

### Key Storage
- **Never commit API keys to version control**
- **Use environment variables** to store keys
- **Use secure secret management** for production
- **Rotate keys regularly** (every 90 days recommended)

### Key Usage
```javascript
// ❌ Don't do this
const apiKey = 'sk_live_1234567890abcdef';

// ✅ Do this instead
const apiKey = process.env.SMARTCOMEX_API_KEY;
```

### Network Security
- **Always use HTTPS** for API requests
- **Implement proper certificate validation**
- **Use secure connections** in production
- **Monitor for suspicious activity**

## Authentication Methods

### Bearer Token (Recommended)
```http
GET /v1/products HTTP/1.1
Host: api.smartcomex.com
Authorization: Bearer sk_live_1234567890abcdef
```

### Query Parameter (Not Recommended)
```http
GET /v1/products?api_key=sk_live_1234567890abcdef HTTP/1.1
Host: api.smartcomex.com
```

### Basic Authentication (Legacy)
```http
GET /v1/products HTTP/1.1
Host: api.smartcomex.com
Authorization: Basic c2tfbGl2ZV8xMjM0NTY3ODkwYWJjZGVm
```

## Error Responses

### Invalid API Key
```json
{
  "success": false,
  "error": {
    "code": "INVALID_API_KEY",
    "message": "The provided API key is invalid or expired",
    "details": {
      "key_id": "sk_live_invalid"
    }
  }
}
```

### Insufficient Permissions
```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "The API key does not have permission for this operation",
    "details": {
      "required_permission": "write",
      "key_permissions": ["read"]
    }
  }
}
```

### Rate Limit Exceeded
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "API rate limit exceeded",
    "details": {
      "limit": 1000,
      "remaining": 0,
      "reset_time": "2024-01-15T10:30:00Z"
    }
  }
}
```

## Testing Authentication

### Verify API Key
Use the authentication endpoint to verify your API key:

```bash
curl -H "Authorization: Bearer your_api_key" \
     https://api.smartcomex.com/v1/auth/verify
```

**Success Response:**
```json
{
  "success": true,
  "data": {
    "key_id": "sk_live_1234567890abcdef",
    "permissions": ["read", "write"],
    "account_id": "acc_987654321",
    "created_at": "2024-01-01T00:00:00Z",
    "last_used": "2024-01-15T09:30:00Z"
  }
}
```

## SDK Authentication

### JavaScript/Node.js
```javascript
const SmartComex = require('@smartcomex/node-sdk');

const client = new SmartComex(process.env.SMARTCOMEX_API_KEY);

// The SDK handles authentication automatically
const products = await client.products.list();
```

### Python
```python
import smartcomex
import os

client = smartcomex.Client(os.environ['SMARTCOMEX_API_KEY'])

# Authentication is handled by the SDK
products = client.products.list()
```

### PHP
```php
use SmartComex\Client;

$client = new Client($_ENV['SMARTCOMEX_API_KEY']);

// SDK manages authentication headers
$products = $client->products->list();
```

## Webhook Authentication

### Webhook Signatures
Webhooks include a signature header to verify authenticity:

```http
X-SmartComex-Signature: sha256=d4b8f8b1a2c3d4e5f6a7b8c9d0e1f2a3
```

### Verifying Webhooks
```javascript
const crypto = require('crypto');

function verifyWebhook(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload, 'utf8')
    .digest('hex');
    
  return `sha256=${expectedSignature}` === signature;
}

// Usage
const isValid = verifyWebhook(
  req.body,
  req.headers['x-smartcomex-signature'],
  process.env.WEBHOOK_SECRET
);
```

## OAuth 2.0 (Enterprise)

For enterprise integrations, SmartComex supports OAuth 2.0:

### Authorization Code Flow
1. **Redirect user to authorization URL**
2. **User authorizes your application**
3. **Receive authorization code**
4. **Exchange code for access token**

### Implementation Example
```javascript
// Step 1: Redirect to authorization URL
const authUrl = `https://api.smartcomex.com/oauth/authorize?` +
  `client_id=${clientId}&` +
  `redirect_uri=${redirectUri}&` +
  `response_type=code&` +
  `scope=read write`;

// Step 2: Exchange code for token
const tokenResponse = await fetch('https://api.smartcomex.com/oauth/token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    grant_type: 'authorization_code',
    client_id: clientId,
    client_secret: clientSecret,
    code: authorizationCode,
    redirect_uri: redirectUri
  })
});

const { access_token } = await tokenResponse.json();
```

## Troubleshooting

### Common Issues

**"Authentication failed"**
- Verify API key is correct and not expired
- Check that the key is properly formatted
- Ensure you're using the right environment (test vs live)

**"Insufficient permissions"**
- Check API key permissions in dashboard
- Verify the key has the required access level
- Contact support if permissions seem incorrect

**"Rate limit exceeded"**
- Implement exponential backoff in your requests
- Consider upgrading your plan for higher limits
- Distribute requests over time

### Debug Mode
Enable debug mode to see authentication details:

```bash
curl -H "Authorization: Bearer your_api_key" \
     -H "X-Debug: true" \
     https://api.smartcomex.com/v1/products
```

## Migration Guide

### From API Key to OAuth
For applications moving from API keys to OAuth:

1. **Register your application** in the developer console
2. **Implement OAuth flow** in your application
3. **Test with sandbox environment**
4. **Migrate production traffic gradually**
5. **Deprecate old API keys** after migration

## Support

For authentication issues:
- Check our [troubleshooting guide](../support/troubleshooting.md)
- Review [API documentation](overview.md)
- Contact [technical support](../support/contact.md)

Remember: Never share your API keys publicly or commit them to version control. Keep them secure and rotate them regularly for optimal security.