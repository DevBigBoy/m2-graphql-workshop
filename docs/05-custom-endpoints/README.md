# Creating Custom GraphQL Endpoints

## Overview

Learn how to create your own custom GraphQL queries and mutations in Magento 2.

## Topics Covered

1. [Module Structure](01-module-structure.md)
2. [Creating a Simple Query](02-simple-query.md)
3. [Creating a Query with Arguments](03-query-with-arguments.md)
4. [Creating a Mutation](04-simple-mutation.md)
5. [Input Types](05-input-types.md)
6. [Output Types](06-output-types.md)
7. [Extending Existing Types](07-extending-types.md)
8. [Complex Nested Resolvers](08-nested-resolvers.md)
9. [Filtering and Pagination](09-filtering-pagination.md)
10. [Complete Example: Blog Module](10-blog-example.md)

## Learning Objectives

By the end of this section, you will be able to:

- Create a custom Magento module with GraphQL support
- Define custom queries and mutations
- Create resolvers to handle data fetching
- Use input and output types
- Extend existing Magento GraphQL types
- Implement filtering and pagination
- Build a complete custom GraphQL API

## Quick Start: Simple Query

### Step 1: Create Module Structure

```text
app/code/Vendor/CustomGraphQL/
├── etc/
│   ├── module.xml
│   └── schema.graphqls
├── Model/
│   └── Resolver/
│       └── HelloWorld.php
├── registration.php
└── composer.json
```

### Step 2: Define Schema (etc/schema.graphqls)

```graphql
type Query {
    helloWorld: String @resolver(class: "Vendor\\CustomGraphQL\\Model\\Resolver\\HelloWorld") @doc(description: "Returns a hello world message")
}
```

### Step 3: Create Resolver

```php
<?php
namespace Vendor\CustomGraphQL\Model\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Query\ResolverInterface;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;

class HelloWorld implements ResolverInterface
{
    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) {
        return "Hello, GraphQL World!";
    }
}
```

### Step 4: Test Your Query

```graphql
{
  helloWorld
}
```

Expected response:

```json
{
  "data": {
    "helloWorld": "Hello, GraphQL World!"
  }
}
```

## Common Patterns

### Query with Arguments

```graphql
type Query {
    customProduct(sku: String!): CustomProductOutput
        @resolver(class: "Vendor\\Module\\Model\\Resolver\\CustomProduct")
}
```

### Mutation

```graphql
type Mutation {
    createCustomEntity(input: CustomEntityInput!): CustomEntityOutput
        @resolver(class: "Vendor\\Module\\Model\\Resolver\\CreateCustomEntity")
}

input CustomEntityInput {
    name: String!
    description: String
    status: Int
}

type CustomEntityOutput {
    entity_id: Int
    name: String
    success: Boolean
    message: String
}
```

## Next Steps

Start with [Module Structure](01-module-structure.md) to learn how to set up your custom module.

---

[← Previous: Built-in Functionality](../04-built-in-functionality/README.md) | [Back to Main](../../README.md) | [Next: Resolvers Deep Dive →](../06-resolvers/README.md)
