# SmartComex API Overview

The SmartComex API provides programmatic access to all platform functionality, allowing you to integrate SmartComex with your existing systems and create custom applications.

## API Characteristics

### RESTful Design
- Uses standard HTTP methods (GET, POST, PUT, DELETE)
- Predictable URL patterns
- JSON request and response format
- HTTP status codes for response indication

### Rate Limiting
- 1000 requests per hour for standard plans
- 5000 requests per hour for premium plans
- Rate limit headers included in responses
- Exponential backoff recommended for retries

### Versioning
- Current version: v1
- Version specified in URL path: `/api/v1/`
- Backward compatibility maintained for major versions
- Deprecation notices provided for upcoming changes

## Base URL

```
Production: https://api.smartcomex.com/v1
Sandbox: https://sandbox-api.smartcomex.com/v1
```

## Authentication

All API requests require authentication using API keys. See [Authentication Guide](authentication.md) for detailed information.

### Quick Example
```bash
curl -H "Authorization: Bearer your_api_key" \
     https://api.smartcomex.com/v1/products
```

## Core Resources

### Products
Manage your product catalog programmatically.

- `GET /products` - List all products
- `POST /products` - Create a new product
- `GET /products/{id}` - Get specific product
- `PUT /products/{id}` - Update a product
- `DELETE /products/{id}` - Delete a product

### Orders
Process and manage customer orders.

- `GET /orders` - List all orders
- `POST /orders` - Create a new order
- `GET /orders/{id}` - Get specific order
- `PUT /orders/{id}` - Update order status
- `POST /orders/{id}/refund` - Process refund

### Customers
Manage customer information and relationships.

- `GET /customers` - List all customers
- `POST /customers` - Create a new customer
- `GET /customers/{id}` - Get specific customer
- `PUT /customers/{id}` - Update customer information
- `GET /customers/{id}/orders` - Get customer orders

### Inventory
Track and manage stock levels.

- `GET /inventory` - List inventory levels
- `PUT /inventory/{product_id}` - Update stock level
- `GET /inventory/alerts` - Get low stock alerts
- `POST /inventory/adjustments` - Record stock adjustments

## Response Format

### Success Response
```json
{
  "success": true,
  "data": {
    // Resource data
  },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request is invalid",
    "details": {
      "field": "email",
      "message": "Email address is required"
    }
  }
}
```

## Pagination

Large result sets are paginated using cursor-based pagination:

```bash
GET /products?page=1&per_page=20
```

Response includes pagination metadata:
```json
{
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_previous": false
  }
}
```

## Filtering and Sorting

### Filtering
```bash
GET /products?category=electronics&status=active
GET /orders?status=completed&created_after=2023-01-01
```

### Sorting
```bash
GET /products?sort=name&order=asc
GET /orders?sort=created_at&order=desc
```

### Field Selection
```bash
GET /products?fields=id,name,price
```

## Webhooks

SmartComex can send webhooks for real-time event notifications:

### Supported Events
- `order.created` - New order placed
- `order.updated` - Order status changed
- `payment.completed` - Payment processed
- `inventory.low_stock` - Stock below threshold
- `customer.created` - New customer registered

### Webhook Configuration
```bash
POST /webhooks
{
  "url": "https://your-app.com/webhook",
  "events": ["order.created", "payment.completed"],
  "secret": "your_webhook_secret"
}
```

## SDK and Libraries

Official SDKs available for popular programming languages:

### JavaScript/Node.js
```bash
npm install @smartcomex/node-sdk
```

```javascript
const SmartComex = require('@smartcomex/node-sdk');
const client = new SmartComex('your_api_key');

const products = await client.products.list();
```

### Python
```bash
pip install smartcomex-python
```

```python
from smartcomex import SmartComex

client = SmartComex('your_api_key')
products = client.products.list()
```

### PHP
```bash
composer require smartcomex/php-sdk
```

```php
use SmartComex\Client;

$client = new Client('your_api_key');
$products = $client->products->list();
```

## Error Handling

### HTTP Status Codes
- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `429` - Rate Limited
- `500` - Internal Server Error

### Error Codes
- `INVALID_REQUEST` - Request validation failed
- `UNAUTHORIZED` - Invalid or missing API key
- `RESOURCE_NOT_FOUND` - Requested resource doesn't exist
- `RATE_LIMITED` - Too many requests
- `INSUFFICIENT_PERMISSIONS` - Access denied

## Best Practices

### API Key Security
- Store API keys securely (environment variables)
- Use different keys for different environments
- Rotate keys regularly
- Monitor API key usage

### Request Optimization
- Use field selection to reduce response size
- Implement proper pagination
- Cache responses when appropriate
- Use bulk operations when available

### Error Handling
- Implement retry logic with exponential backoff
- Handle rate limiting gracefully
- Log errors for debugging
- Provide user-friendly error messages

### Testing
- Use sandbox environment for development
- Test error scenarios
- Validate webhook handling
- Monitor API performance

## Examples

### Creating a Product
```bash
curl -X POST https://api.smartcomex.com/v1/products \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sample Product",
    "description": "A sample product description",
    "price": 29.99,
    "sku": "SAMPLE-001",
    "category": "electronics",
    "inventory": {
      "quantity": 100,
      "track_inventory": true
    }
  }'
```

### Updating Order Status
```bash
curl -X PUT https://api.smartcomex.com/v1/orders/12345 \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "shipped",
    "tracking_number": "1Z999AA1234567890",
    "notes": "Order shipped via UPS"
  }'
```

### Fetching Customer Orders
```bash
curl -H "Authorization: Bearer your_api_key" \
     "https://api.smartcomex.com/v1/customers/67890/orders?status=completed"
```

## Rate Limiting

### Rate Limit Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### Handling Rate Limits
```javascript
const makeRequest = async (url, options) => {
  try {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      const resetTime = response.headers.get('X-RateLimit-Reset');
      const delay = (resetTime * 1000) - Date.now();
      
      await new Promise(resolve => setTimeout(resolve, delay));
      return makeRequest(url, options); // Retry
    }
    
    return response;
  } catch (error) {
    console.error('API request failed:', error);
    throw error;
  }
};
```

## Support

### API Documentation
- Complete endpoint reference: [Endpoints](endpoints.md)
- Authentication guide: [Authentication](authentication.md)
- SDK documentation: [SDKs](sdks.md)

### Getting Help
- API Support: api-support@smartcomex.com
- Developer Forum: https://developers.smartcomex.com
- GitHub Issues: https://github.com/smartcomex/api-issues

### Status and Updates
- API Status: https://status.smartcomex.com
- Changelog: https://developers.smartcomex.com/changelog
- Breaking Changes: 30-day advance notice via email

## Changelog

### v1.2.0 (2024-01-15)
- Added bulk product operations
- Improved error message details
- New webhook events for inventory

### v1.1.0 (2023-12-01)
- Added customer groups endpoint
- Enhanced filtering options
- Performance improvements

### v1.0.0 (2023-10-01)
- Initial API release
- Core CRUD operations for all resources
- Webhook support