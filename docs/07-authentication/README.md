# Authentication & Authorization

## Overview

Learn how to secure your GraphQL endpoints with proper authentication and authorization mechanisms.

## Topics Covered

1. [Authentication Overview](01-overview.md)
2. [Customer Token Authentication](02-customer-tokens.md)
   - Generating tokens
   - Token structure
   - Token expiration
   - Sending tokens in requests
3. [Admin Token Authentication](03-admin-tokens.md)
4. [Integration Tokens](04-integration-tokens.md)
5. [OAuth Authentication](05-oauth.md)
6. [Authorization in Resolvers](06-authorization.md)
   - Checking user context
   - Role-based access control
   - Custom authorization logic
7. [Guest vs Authenticated Flows](07-guest-flows.md)
8. [Token Management](08-token-management.md)
   - Refresh strategies
   - Revoke tokens
   - Token storage best practices
9. [Security Best Practices](09-security-practices.md)

## Learning Objectives

By the end of this section, you will:

- Understand different authentication methods
- Know how to generate and use customer tokens
- Implement authorization checks in resolvers
- Handle guest vs authenticated user flows
- Manage token lifecycle
- Follow security best practices

## Authentication Flow

### Step 1: Generate Customer Token

```graphql
mutation {
  generateCustomerToken(
    email: "customer@example.com"
    password: "SecurePassword123"
  ) {
    token
  }
}
```

Response:

```json
{
  "data": {
    "generateCustomerToken": {
      "token": "eyJraWQiOiIxIiwiYWxnIjoiSFMyNTYifQ..."
    }
  }
}
```

### Step 2: Use Token in Requests

Add the token to the Authorization header:

```http
POST /graphql HTTP/1.1
Host: your-magento-store.com
Content-Type: application/json
Authorization: Bearer eyJraWQiOiIxIiwiYWxnIjoiSFMyNTYifQ...

{
  "query": "{ customer { firstname lastname email } }"
}
```

### Step 3: Access Protected Resources

```graphql
{
  customer {
    firstname
    lastname
    email
    addresses {
      street
      city
      postcode
    }
  }
}
```

## Authorization in Resolvers

```php
<?php
namespace Vendor\Module\Model\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Query\ResolverInterface;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;
use Magento\Framework\GraphQl\Exception\GraphQlAuthorizationException;

class MyProtectedResolver implements ResolverInterface
{
    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) {
        // Check if user is authenticated
        if (!$context->getUserId()) {
            throw new GraphQlAuthorizationException(
                __('The current customer isn\'t authorized.')
            );
        }

        // Get customer ID
        $customerId = $context->getUserId();

        // Your logic here
        return $this->getCustomerData($customerId);
    }
}
```

## Context Methods

The `$context` object provides:

```php
// Get current user/customer ID
$userId = $context->getUserId();

// Get user type (customer or admin)
$userType = $context->getUserType();

// Get store ID
$storeId = $context->getExtensionAttributes()->getStore()->getId();

// Check if user is logged in
if ($context->getUserId()) {
    // User is authenticated
}
```

## Guest Cart Example

```graphql
# Create guest cart
mutation {
  createEmptyCart
}

# Add product to guest cart
mutation {
  addSimpleProductsToCart(
    input: {
      cart_id: "guest_cart_id_here"
      cart_items: [
        {
          data: {
            quantity: 1
            sku: "24-MB01"
          }
        }
      ]
    }
  ) {
    cart {
      items {
        id
        product {
          name
          sku
        }
        quantity
      }
    }
  }
}
```

## Token Configuration

Token expiration can be configured in:

**Admin Panel**: Stores > Configuration > Services > OAuth > Access Token Expiration

Or via CLI:

```bash
bin/magento config:set oauth/access_token_lifetime/customer 3600
```

## Security Checklist

- ✅ Always validate user authentication for protected resources
- ✅ Use HTTPS in production
- ✅ Implement proper token expiration
- ✅ Never expose tokens in URLs
- ✅ Validate user permissions for specific actions
- ✅ Implement rate limiting
- ✅ Log authentication attempts
- ✅ Use strong password policies

## Next Steps

Start with [Authentication Overview](01-overview.md) to understand the authentication mechanisms.

---

[← Previous: Resolvers](../06-resolvers/README.md) | [Back to Main](../../README.md) | [Next: Error Handling →](../08-error-handling/README.md)
