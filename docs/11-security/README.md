# Security Best Practices

## Overview

Security is critical for production GraphQL APIs. This section covers essential security practices and common vulnerabilities.

## Topics Covered

1. [Security Overview](01-overview.md)
2. [Input Validation](02-input-validation.md)
3. [Output Sanitization](03-output-sanitization.md)
4. [Authentication Security](04-authentication-security.md)
5. [Authorization Best Practices](05-authorization.md)
6. [Rate Limiting](06-rate-limiting.md)
7. [Query Complexity Limiting](07-query-complexity.md)
8. [CORS Configuration](08-cors.md)
9. [Preventing Common Attacks](09-common-attacks.md)
   - SQL Injection
   - XSS (Cross-Site Scripting)
   - CSRF
   - Information Disclosure
10. [SSL/TLS Configuration](10-ssl-tls.md)
11. [Security Headers](11-security-headers.md)
12. [Logging and Monitoring](12-logging-monitoring.md)
13. [Security Checklist](13-checklist.md)

## Learning Objectives

By the end of this section, you will:

- Implement proper input validation
- Secure authentication and authorization
- Prevent common web vulnerabilities
- Configure rate limiting
- Set up security monitoring
- Follow production security best practices

## Critical Security Rules

### 1. Always Use HTTPS in Production

```nginx
# Nginx configuration
server {
    listen 443 ssl http2;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
}
```

### 2. Validate All Inputs

```php
<?php
public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
{
    // Validate required fields
    if (!isset($args['email']) || empty($args['email'])) {
        throw new GraphQlInputException(__('Email is required'));
    }

    // Validate email format
    if (!filter_var($args['email'], FILTER_VALIDATE_EMAIL)) {
        throw new GraphQlInputException(__('Invalid email format'));
    }

    // Validate input length
    if (strlen($args['name']) > 255) {
        throw new GraphQlInputException(__('Name is too long'));
    }

    // Sanitize input
    $email = filter_var($args['email'], FILTER_SANITIZE_EMAIL);

    // Continue processing...
}
```

### 3. Implement Proper Authorization

```php
<?php
public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
{
    // Check authentication
    if (!$context->getUserId()) {
        throw new GraphQlAuthenticationException(
            __('Current user is not authenticated')
        );
    }

    // Check authorization for specific resource
    $customerId = $context->getUserId();
    $requestedId = $args['customer_id'];

    if ($customerId != $requestedId) {
        throw new GraphQlAuthorizationException(
            __('You are not authorized to access this resource')
        );
    }

    // Continue processing...
}
```

### 4. Implement Rate Limiting

```xml
<!-- etc/di.xml -->
<type name="Magento\GraphQl\Controller\GraphQl">
    <plugin name="rate_limiter" type="Vendor\Module\Plugin\RateLimiter"/>
</type>
```

```php
<?php
namespace Vendor\Module\Plugin;

class RateLimiter
{
    private $cache;
    private const RATE_LIMIT = 100; // requests per minute
    private const RATE_PERIOD = 60; // seconds

    public function beforeDispatch(
        \Magento\GraphQl\Controller\GraphQl $subject,
        \Magento\Framework\App\RequestInterface $request
    ) {
        $identifier = $this->getClientIdentifier($request);
        $cacheKey = 'graphql_rate_limit_' . $identifier;

        $requestCount = $this->cache->load($cacheKey) ?: 0;

        if ($requestCount >= self::RATE_LIMIT) {
            throw new \Magento\Framework\Exception\LocalizedException(
                __('Rate limit exceeded. Please try again later.')
            );
        }

        $this->cache->save($requestCount + 1, $cacheKey, [], self::RATE_PERIOD);
    }

    private function getClientIdentifier($request): string
    {
        // Use IP address or customer ID
        return $request->getClientIp();
    }
}
```

### 5. Limit Query Complexity

```php
<?php
namespace Vendor\Module\Plugin;

use Magento\Framework\GraphQl\Exception\GraphQlInputException;

class QueryComplexityValidator
{
    private const MAX_QUERY_DEPTH = 10;
    private const MAX_QUERY_COMPLEXITY = 100;

    public function beforeDispatch($subject, $request)
    {
        $query = $this->getQueryFromRequest($request);

        if ($this->getQueryDepth($query) > self::MAX_QUERY_DEPTH) {
            throw new GraphQlInputException(
                __('Query is too deep. Maximum depth is %1', self::MAX_QUERY_DEPTH)
            );
        }

        if ($this->getQueryComplexity($query) > self::MAX_QUERY_COMPLEXITY) {
            throw new GraphQlInputException(
                __('Query is too complex')
            );
        }
    }
}
```

### 6. Sanitize Output

```php
<?php
public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
{
    $data = $this->dataProvider->getData();

    // Sanitize output to prevent XSS
    return [
        'name' => htmlspecialchars($data->getName(), ENT_QUOTES, 'UTF-8'),
        'description' => strip_tags($data->getDescription()),
        'email' => filter_var($data->getEmail(), FILTER_SANITIZE_EMAIL)
    ];
}
```

### 7. Never Expose Sensitive Data

```php
<?php
// BAD - Exposes sensitive information
public function resolve(...)
{
    return [
        'customer' => [
            'password_hash' => $customer->getPasswordHash(), // NEVER DO THIS
            'api_key' => $customer->getApiKey(), // NEVER DO THIS
        ]
    ];
}

// GOOD - Only expose necessary data
public function resolve(...)
{
    return [
        'customer' => [
            'firstname' => $customer->getFirstname(),
            'lastname' => $customer->getLastname(),
            'email' => $customer->getEmail()
        ]
    ];
}
```

### 8. Prevent SQL Injection

```php
<?php
// BAD - Vulnerable to SQL injection
$sql = "SELECT * FROM products WHERE sku = '" . $args['sku'] . "'";

// GOOD - Use prepared statements or repositories
$searchCriteria = $this->searchCriteriaBuilder
    ->addFilter('sku', $args['sku'], 'eq')
    ->create();
$products = $this->productRepository->getList($searchCriteria);
```

### 9. Implement CORS Properly

```xml
<!-- etc/di.xml -->
<type name="Magento\GraphQl\Controller\GraphQl">
    <plugin name="cors_headers" type="Vendor\Module\Plugin\CorsHeaders"/>
</type>
```

```php
<?php
namespace Vendor\Module\Plugin;

class CorsHeaders
{
    public function afterDispatch($subject, $result, $request)
    {
        $response = $result;

        // Only allow specific origins
        $allowedOrigins = [
            'https://your-frontend.com',
            'https://your-app.com'
        ];

        $origin = $request->getHeader('Origin');

        if (in_array($origin, $allowedOrigins)) {
            $response->setHeader('Access-Control-Allow-Origin', $origin);
            $response->setHeader('Access-Control-Allow-Methods', 'POST, GET, OPTIONS');
            $response->setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
            $response->setHeader('Access-Control-Max-Age', '86400');
        }

        return $response;
    }
}
```

### 10. Log Security Events

```php
<?php
private function logSecurityEvent($event, $context = [])
{
    $this->logger->warning('GraphQL Security Event: ' . $event, [
        'ip' => $this->request->getClientIp(),
        'user_id' => $this->context->getUserId(),
        'timestamp' => time(),
        'context' => $context
    ]);
}

// Usage
if (!$this->isAuthorized()) {
    $this->logSecurityEvent('Unauthorized access attempt', [
        'resource' => $resourceId,
        'action' => 'read'
    ]);
    throw new GraphQlAuthorizationException(__('Not authorized'));
}
```

## Security Checklist

### Authentication & Authorization

- ✅ Use HTTPS only in production
- ✅ Implement proper token-based authentication
- ✅ Set appropriate token expiration
- ✅ Validate user permissions for all protected resources
- ✅ Never trust client input
- ✅ Implement role-based access control

### Input Validation

- ✅ Validate all inputs
- ✅ Use type validation
- ✅ Implement length limits
- ✅ Sanitize user input
- ✅ Validate email formats
- ✅ Check for required fields

### Rate Limiting & Complexity

- ✅ Implement rate limiting per IP/user
- ✅ Limit query depth
- ✅ Limit query complexity
- ✅ Set maximum batch size
- ✅ Implement timeout limits

### Data Protection

- ✅ Never expose password hashes
- ✅ Never expose API keys
- ✅ Filter sensitive data from responses
- ✅ Sanitize output to prevent XSS
- ✅ Use parameterized queries
- ✅ Encrypt sensitive data at rest

### Monitoring & Logging

- ✅ Log authentication attempts
- ✅ Log authorization failures
- ✅ Monitor for suspicious patterns
- ✅ Set up alerts for security events
- ✅ Regular security audits
- ✅ Monitor query performance

### Infrastructure

- ✅ Use Web Application Firewall (WAF)
- ✅ Enable DDoS protection
- ✅ Keep Magento and dependencies updated
- ✅ Regular security patches
- ✅ Secure server configuration
- ✅ Database access restrictions

## Common Vulnerabilities to Avoid

### 1. Broken Authentication

```php
// BAD - Weak password validation
if (strlen($password) >= 6) {
    // Create account
}

// GOOD - Strong password requirements
if (!preg_match('/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/', $password)) {
    throw new GraphQlInputException(__('Password must be at least 8 characters with uppercase, lowercase, number, and special character'));
}
```

### 2. Information Disclosure

```php
// BAD - Exposes internal details
catch (\Exception $e) {
    throw new GraphQlInputException(__($e->getMessage()));
}

// GOOD - Generic error message, log details
catch (\Exception $e) {
    $this->logger->error($e->getMessage());
    throw new GraphQlInputException(__('An error occurred processing your request'));
}
```

### 3. Insufficient Logging

```php
// BAD - No logging
if (!$this->authorize($user, $resource)) {
    throw new GraphQlAuthorizationException(__('Not authorized'));
}

// GOOD - Log security events
if (!$this->authorize($user, $resource)) {
    $this->securityLogger->warning('Authorization failed', [
        'user_id' => $user->getId(),
        'resource' => $resource,
        'ip' => $this->getClientIp()
    ]);
    throw new GraphQlAuthorizationException(__('Not authorized'));
}
```

## Next Steps

Start with [Security Overview](01-overview.md) for a comprehensive security introduction.

---

[← Previous: Testing](../10-testing/README.md) | [Back to Main](../../README.md) | [Next: Debugging →](../12-debugging/README.md)
