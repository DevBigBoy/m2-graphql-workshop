# Deep Dive: Resolvers

## Overview

Resolvers are the heart of GraphQL implementation. This section provides comprehensive coverage of resolver patterns and best practices.

## Topics Covered

1. [Resolver Interface](01-resolver-interface.md)
2. [Understanding Resolver Parameters](02-resolver-parameters.md)
   - Field
   - Context
   - ResolveInfo
   - Value
   - Args
3. [Context Object](03-context-object.md)
   - User context
   - Store context
   - Extension attributes
4. [ResolveInfo Object](04-resolve-info.md)
   - Field selection
   - Optimizing based on requested fields
   - Parent type information
5. [Resolver Types](05-resolver-types.md)
   - Field resolvers
   - Type resolvers
   - Collection resolvers
   - Value resolvers
6. [Data Loading Patterns](06-data-loading.md)
   - Single entity loading
   - Collection loading
   - Batch loading (DataLoader pattern)
7. [Avoiding N+1 Queries](07-n-plus-one.md)
8. [Resolver Best Practices](08-best-practices.md)
9. [Performance Optimization](09-performance.md)
10. [Testing Resolvers](10-testing-resolvers.md)

## Learning Objectives

By the end of this section, you will:

- Understand the ResolverInterface in depth
- Know how to use all resolver parameters effectively
- Implement efficient data loading patterns
- Avoid common performance pitfalls
- Write testable, maintainable resolvers
- Follow Magento best practices

## Resolver Interface

```php
<?php
namespace Magento\Framework\GraphQl\Query;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;

interface ResolverInterface
{
    /**
     * Resolve field value
     *
     * @param Field $field
     * @param \Magento\Framework\GraphQl\Query\Resolver\ContextInterface $context
     * @param ResolveInfo $info
     * @param array|null $value
     * @param array|null $args
     * @return mixed|\Magento\Framework\GraphQl\Query\Resolver\Value
     */
    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    );
}
```

## Resolver Parameters Explained

### 1. Field ($field)

Contains metadata about the field being resolved:

- Field name
- Field type
- Field arguments

### 2. Context ($context)

Provides access to:

- Current user/customer information
- Store information
- Extension attributes
- Custom context data

### 3. ResolveInfo ($info)

Contains information about:

- Query structure
- Requested fields
- Field selection set
- Parent type

### 4. Value ($value)

Parent value from previous resolver (for nested fields)

### 5. Args ($args)

Arguments passed to the field in the query

## Example: Complete Resolver

```php
<?php
namespace Vendor\Module\Model\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Query\ResolverInterface;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;
use Magento\Framework\GraphQl\Exception\GraphQlInputException;

class ProductByCategory implements ResolverInterface
{
    private $productRepository;
    private $searchCriteriaBuilder;

    public function __construct(
        \Magento\Catalog\Api\ProductRepositoryInterface $productRepository,
        \Magento\Framework\Api\SearchCriteriaBuilder $searchCriteriaBuilder
    ) {
        $this->productRepository = $productRepository;
        $this->searchCriteriaBuilder = $searchCriteriaBuilder;
    }

    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) {
        // Validate arguments
        if (!isset($args['category_id'])) {
            throw new GraphQlInputException(__('Category ID is required'));
        }

        // Build search criteria
        $searchCriteria = $this->searchCriteriaBuilder
            ->addFilter('category_id', $args['category_id'], 'eq')
            ->create();

        // Fetch data
        $searchResult = $this->productRepository->getList($searchCriteria);

        // Return formatted data
        return [
            'items' => $searchResult->getItems(),
            'total_count' => $searchResult->getTotalCount()
        ];
    }
}
```

## Best Practices

1. **Keep resolvers thin** - Delegate business logic to service classes
2. **Use dependency injection** - Don't instantiate objects directly
3. **Validate input** - Always validate arguments
4. **Handle errors gracefully** - Use GraphQL exceptions
5. **Optimize queries** - Avoid N+1 problems
6. **Use type hints** - For better code quality
7. **Add documentation** - Use PHPDoc comments

## Next Steps

Start with [Resolver Interface](01-resolver-interface.md) to understand the basics.

---

[← Previous: Custom Endpoints](../05-custom-endpoints/README.md) | [Back to Main](../../README.md) | [Next: Authentication →](../07-authentication/README.md)
