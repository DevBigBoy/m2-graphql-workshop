# Error Handling

## Overview

Proper error handling is crucial for building robust GraphQL APIs. This section covers error types, best practices, and implementation strategies.

## Topics Covered

1. [GraphQL Error Structure](01-error-structure.md)
2. [Error Types](02-error-types.md)
   - Syntax errors
   - Validation errors
   - Execution errors
   - Application errors
3. [Magento GraphQL Exceptions](03-magento-exceptions.md)
   - GraphQlInputException
   - GraphQlAuthorizationException
   - GraphQlAuthenticationException
   - GraphQlNoSuchEntityException
4. [Custom Exceptions](04-custom-exceptions.md)
5. [Error Response Format](05-error-response.md)
6. [Partial Data Responses](06-partial-data.md)
7. [Error Logging](07-error-logging.md)
8. [Client-Side Error Handling](08-client-side.md)
9. [Best Practices](09-best-practices.md)

## Learning Objectives

By the end of this section, you will:

- Understand GraphQL error structure
- Know when to use different exception types
- Create custom exceptions
- Implement proper error handling in resolvers
- Log errors effectively
- Provide meaningful error messages
- Handle errors on the client side

## GraphQL Error Response Structure

```json
{
  "errors": [
    {
      "message": "The product that was requested doesn't exist.",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": ["products", "items", 0],
      "extensions": {
        "category": "graphql-no-such-entity"
      }
    }
  ],
  "data": {
    "products": null
  }
}
```

## Error Components

### 1. message

Human-readable error description

### 2. locations

Where in the query the error occurred (line and column)

### 3. path

Path to the field that caused the error

### 4. extensions

Additional error metadata (category, debug info, etc.)

## Magento Exception Classes

```php
use Magento\Framework\GraphQl\Exception\GraphQlInputException;
use Magento\Framework\GraphQl\Exception\GraphQlAuthorizationException;
use Magento\Framework\GraphQl\Exception\GraphQlAuthenticationException;
use Magento\Framework\GraphQl\Exception\GraphQlNoSuchEntityException;

// Input validation error
throw new GraphQlInputException(__('Invalid SKU provided'));

// Authorization error (user not allowed)
throw new GraphQlAuthorizationException(
    __('The current customer isn\'t authorized to perform this action.')
);

// Authentication error (not logged in)
throw new GraphQlAuthenticationException(
    __('The current customer isn\'t authenticated.')
);

// Entity not found
throw new GraphQlNoSuchEntityException(
    __('The product with SKU "%1" doesn\'t exist.', $sku)
);
```

## Error Handling in Resolvers

```php
<?php
namespace Vendor\Module\Model\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Query\ResolverInterface;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;
use Magento\Framework\GraphQl\Exception\GraphQlInputException;
use Magento\Framework\GraphQl\Exception\GraphQlNoSuchEntityException;
use Psr\Log\LoggerInterface;

class ProductBySku implements ResolverInterface
{
    private $productRepository;
    private $logger;

    public function __construct(
        \Magento\Catalog\Api\ProductRepositoryInterface $productRepository,
        LoggerInterface $logger
    ) {
        $this->productRepository = $productRepository;
        $this->logger = $logger;
    }

    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) {
        // Validate input
        if (!isset($args['sku']) || empty($args['sku'])) {
            throw new GraphQlInputException(__('SKU is required'));
        }

        try {
            // Attempt to load product
            $product = $this->productRepository->get($args['sku']);

            return [
                'id' => $product->getId(),
                'sku' => $product->getSku(),
                'name' => $product->getName()
            ];

        } catch (\Magento\Framework\Exception\NoSuchEntityException $e) {
            // Entity not found
            throw new GraphQlNoSuchEntityException(
                __('Product with SKU "%1" not found', $args['sku'])
            );

        } catch (\Exception $e) {
            // Log unexpected errors
            $this->logger->critical('GraphQL Error: ' . $e->getMessage(), [
                'sku' => $args['sku'],
                'trace' => $e->getTraceAsString()
            ]);

            // Return generic error to client
            throw new GraphQlInputException(
                __('An error occurred while processing your request')
            );
        }
    }
}
```

## Error Categories

Magento uses error categories in the `extensions` field:

- `graphql-input` - Invalid input
- `graphql-authorization` - Authorization failed
- `graphql-authentication` - Authentication required
- `graphql-no-such-entity` - Entity not found
- `internal` - Internal server error

## Best Practices

### 1. Be Specific

```php
// Bad
throw new GraphQlInputException(__('Invalid input'));

// Good
throw new GraphQlInputException(
    __('The email address "%1" is already in use', $email)
);
```

### 2. Don't Expose Sensitive Information

```php
// Bad - exposes internal details
throw new GraphQlInputException(__($e->getMessage()));

// Good - generic message, log details
$this->logger->error($e->getMessage());
throw new GraphQlInputException(__('Unable to process request'));
```

### 3. Log Critical Errors

```php
try {
    // risky operation
} catch (\Exception $e) {
    $this->logger->critical('Critical error in GraphQL', [
        'exception' => $e->getMessage(),
        'trace' => $e->getTraceAsString(),
        'context' => $context
    ]);
    throw new GraphQlInputException(__('An error occurred'));
}
```

### 4. Validate Early

```php
public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
{
    // Validate all inputs first
    $this->validateInput($args);

    // Then process
    return $this->processRequest($args);
}
```

### 5. Use Appropriate Exception Types

```php
// Authentication required
if (!$context->getUserId()) {
    throw new GraphQlAuthenticationException(__('Authentication required'));
}

// User not authorized for action
if (!$this->isAuthorized($context->getUserId(), $resource)) {
    throw new GraphQlAuthorizationException(__('Not authorized'));
}

// Invalid input
if (!$this->validator->isValid($input)) {
    throw new GraphQlInputException(__('Invalid input: %1', $this->validator->getErrors()));
}

// Entity not found
if (!$entity) {
    throw new GraphQlNoSuchEntityException(__('Entity not found'));
}
```

## Partial Data Example

GraphQL can return partial data with errors:

```json
{
  "data": {
    "products": {
      "items": [
        {
          "sku": "24-MB01",
          "name": "Product 1",
          "price": 50.00
        },
        null
      ]
    }
  },
  "errors": [
    {
      "message": "Product with SKU 'invalid-sku' not found",
      "path": ["products", "items", 1]
    }
  ]
}
```

## Next Steps

Start with [GraphQL Error Structure](01-error-structure.md) to understand error formatting.

---

[← Previous: Authentication](../07-authentication/README.md) | [Back to Main](../../README.md) | [Next: Advanced Topics →](../09-advanced-topics/README.md)
