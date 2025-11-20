# Debugging & Troubleshooting

## Overview

Learn effective debugging techniques and solutions to common GraphQL problems in Magento 2.

## Topics Covered

1. [Debugging Tools](01-debugging-tools.md)
2. [Developer Mode](02-developer-mode.md)
3. [GraphQL Query Logging](03-query-logging.md)
4. [Exception Logging](04-exception-logging.md)
5. [Using Xdebug with GraphQL](05-xdebug.md)
6. [Schema Validation Errors](06-schema-errors.md)
7. [Resolver Debugging](07-resolver-debugging.md)
8. [Performance Profiling](08-performance-profiling.md)
9. [Common Errors and Solutions](09-common-errors.md)
10. [Cache Issues](10-cache-issues.md)
11. [Module Loading Order](11-module-loading.md)
12. [Debugging Best Practices](12-best-practices.md)

## Learning Objectives

By the end of this section, you will:

- Enable and use developer mode effectively
- Log and trace GraphQL queries
- Use Xdebug for resolver debugging
- Identify and fix schema errors
- Solve common GraphQL issues
- Profile query performance
- Implement debugging best practices

## Essential Debugging Setup

### 1. Enable Developer Mode

```bash
# Enable developer mode
bin/magento deploy:mode:set developer

# Check current mode
bin/magento deploy:mode:show
```

Benefits of developer mode:

- Detailed error messages
- No static file caching
- Enhanced logging
- Automatic code generation

### 2. Enable GraphQL Logging

```xml
<!-- app/etc/env.php -->
<?php
return [
    'system' => [
        'default' => [
            'dev' => [
                'debug' => [
                    'graphql_log' => 1
                ]
            ]
        ]
    ]
];
```

### 3. Configure Logging

```xml
<!-- etc/di.xml -->
<type name="Vendor\Module\Model\Resolver\YourResolver">
    <arguments>
        <argument name="logger" xsi:type="object">Psr\Log\LoggerInterface</argument>
    </arguments>
</type>
```

```php
<?php
namespace Vendor\Module\Model\Resolver;

use Psr\Log\LoggerInterface;

class YourResolver implements ResolverInterface
{
    private $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
    {
        $this->logger->debug('Resolver called', [
            'args' => $args,
            'user_id' => $context->getUserId(),
            'fields' => $info->getFieldSelection()
        ]);

        // Your logic
    }
}
```

## Common Issues and Solutions

### Issue 1: Schema Not Updating

**Problem**: Changes to `schema.graphqls` not reflected

**Solutions**:

```bash
# Clear cache
bin/magento cache:clean
bin/magento cache:flush

# In developer mode, also regenerate
bin/magento setup:upgrade

# If still not working, clear generated code
rm -rf generated/*
bin/magento setup:di:compile
```

### Issue 2: "The current customer isn't authorized"

**Problem**: Authentication errors even with valid token

**Debug Steps**:

```php
// In your resolver
$this->logger->debug('Auth Debug', [
    'user_id' => $context->getUserId(),
    'user_type' => $context->getUserType(),
    'extension_attributes' => $context->getExtensionAttributes()
]);

// Check token expiration
bin/magento config:show oauth/access_token_lifetime/customer
```

**Solutions**:

- Verify token is not expired
- Check Authorization header format: `Bearer <token>`
- Regenerate customer token
- Verify customer account is active

### Issue 3: "Cannot query field X on type Y"

**Problem**: Field not found in schema

**Debug Steps**:

```bash
# Check if schema file is in correct location
find app/code -name "schema.graphqls"

# Verify module is enabled
bin/magento module:status

# Check module loading order
bin/magento module:status | grep -A5 "List of enabled modules"
```

**Solutions**:

```xml
<!-- Ensure module is enabled -->
bin/magento module:enable Vendor_Module

<!-- Check schema syntax -->
<!-- etc/schema.graphqls -->
type Query {
    myField: String @resolver(class: "Vendor\\Module\\Model\\Resolver\\MyField")
}

<!-- Clear cache -->
bin/magento cache:flush
```

### Issue 4: N+1 Query Problem

**Problem**: Resolver executing too many database queries

**Debug**:

```php
// Enable query logging
<!-- app/etc/env.php -->
'db' => [
    'table_prefix' => '',
    'connection' => [
        'default' => [
            'profiler' => [
                'class' => '\Magento\Framework\DB\Profiler',
                'enabled' => true
            ]
        ]
    ]
]
```

**Solution**:

```php
// BAD - N+1 problem
foreach ($productIds as $id) {
    $product = $this->productRepository->getById($id);
    $products[] = $product;
}

// GOOD - Single query
$searchCriteria = $this->searchCriteriaBuilder
    ->addFilter('entity_id', $productIds, 'in')
    ->create();
$searchResult = $this->productRepository->getList($searchCriteria);
$products = $searchResult->getItems();
```

### Issue 5: Memory Exhausted

**Problem**: Fatal error - Allowed memory size exhausted

**Debug**:

```php
// Log memory usage
$this->logger->debug('Memory usage', [
    'current' => memory_get_usage(true),
    'peak' => memory_get_peak_usage(true)
]);
```

**Solutions**:

```bash
# Increase PHP memory limit
# php.ini
memory_limit = 2G

# For specific script
php -d memory_limit=4G bin/magento ...

# Optimize your resolver
# - Use pagination
# - Limit collection size
# - Clear collections after use
```

```php
// Implement pagination
$searchCriteria = $this->searchCriteriaBuilder
    ->setPageSize($args['pageSize'] ?? 20)
    ->setCurrentPage($args['currentPage'] ?? 1)
    ->create();
```

### Issue 6: CORS Errors

**Problem**: CORS policy blocking requests

**Solution**:

```xml
<!-- etc/di.xml -->
<type name="Magento\GraphQl\Controller\GraphQl">
    <plugin name="graphql_cors" type="Vendor\Module\Plugin\Cors"/>
</type>
```

```php
<?php
namespace Vendor\Module\Plugin;

class Cors
{
    public function beforeDispatch(
        \Magento\GraphQl\Controller\GraphQl $subject,
        \Magento\Framework\App\RequestInterface $request
    ) {
        if ($request->isOptions()) {
            $response = $subject->getResponse();
            $response->setHeader('Access-Control-Allow-Origin', '*');
            $response->setHeader('Access-Control-Allow-Methods', 'POST, GET, OPTIONS');
            $response->setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
            $response->sendResponse();
            exit;
        }
    }
}
```

## Using Xdebug

### 1. Install and Configure Xdebug

```ini
; php.ini
[xdebug]
zend_extension=xdebug.so
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_port=9003
xdebug.client_host=localhost
```

### 2. Set Breakpoints in Resolvers

```php
<?php
namespace Vendor\Module\Model\Resolver;

class ProductResolver implements ResolverInterface
{
    public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
    {
        // Set breakpoint here
        $sku = $args['sku'] ?? null;

        // Debug variables
        $product = $this->productRepository->get($sku);

        return $this->formatProductData($product);
    }
}
```

### 3. Debug GraphQL Request

1. Set breakpoint in your resolver
2. Start Xdebug session
3. Send GraphQL query
4. Step through code

## Query Profiling

```php
<?php
namespace Vendor\Module\Model\Resolver;

use Magento\Framework\Profiler;

class ProductResolver implements ResolverInterface
{
    public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
    {
        Profiler::start('PRODUCT_RESOLVER');

        try {
            // Your logic
            $result = $this->getProducts($args);

            return $result;
        } finally {
            Profiler::stop('PRODUCT_RESOLVER');
        }
    }
}
```

## Debugging Checklist

- ✅ Enable developer mode
- ✅ Clear all caches
- ✅ Check module is enabled
- ✅ Verify schema syntax
- ✅ Check resolver class exists
- ✅ Verify namespace and class name
- ✅ Check di.xml configuration
- ✅ Review exception logs
- ✅ Test query in GraphQL IDE
- ✅ Verify authentication token
- ✅ Check database queries
- ✅ Monitor memory usage
- ✅ Profile performance

## Useful Commands

```bash
# Clear cache
bin/magento cache:clean
bin/magento cache:flush

# Check logs
tail -f var/log/system.log
tail -f var/log/exception.log
tail -f var/log/debug.log

# Recompile
bin/magento setup:di:compile

# Check module status
bin/magento module:status Vendor_Module

# Enable/disable module
bin/magento module:enable Vendor_Module
bin/magento module:disable Vendor_Module

# Upgrade schema
bin/magento setup:upgrade

# Check GraphQL schema
bin/magento dev:query-log:enable
```

## Log File Locations

```text
var/log/system.log       - General system logs
var/log/exception.log    - Exception traces
var/log/debug.log        - Debug messages
var/log/graphql.log      - GraphQL specific logs (if configured)
```

## Next Steps

Start with [Debugging Tools](01-debugging-tools.md) to set up your debugging environment.

---

[← Previous: Security](../11-security/README.md) | [Back to Main](../../README.md) | [Next: Examples →](../13-examples/README.md)
