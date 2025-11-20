# Magento 2 GraphQL Architecture

## Overview

This section provides an in-depth look at how Magento 2 implements GraphQL, including the core components and request flow.

## Topics Covered

1. [GraphQL Architecture Overview](01-overview.md)
2. [Schema Definition](02-schema-definition.md)
   - schema.graphqls file location
   - Type definitions
   - Query and Mutation declarations
   - Input vs Output types
   - Interfaces and Unions
   - Enums and Scalars
3. [Resolvers](03-resolvers.md)
   - ResolverInterface
   - Resolver location and structure
   - Resolver responsibility
4. [Data Providers](04-data-providers.md)
   - Separation of concerns
   - Service layer integration
   - Reusable business logic
5. [Schema Stitching](05-schema-stitching.md)
   - How Magento merges schemas
   - Module dependencies
   - Extending existing types
6. [Request Flow](06-request-flow.md)
   - HTTP request processing
   - Query parsing and validation
   - Resolver execution
   - Response formatting
7. [Dependency Injection](07-dependency-injection.md)
   - di.xml configuration
   - Resolver dependencies
   - Virtual types

## Learning Objectives

By the end of this section, you will understand:

- How Magento organizes GraphQL code
- The role of schema files
- How resolvers work
- Data flow from request to response
- How to extend existing GraphQL types
- Module structure for GraphQL

## Core Components

### 1. Schema (schema.graphqls)

Located in: `app/code/{Vendor}/{Module}/etc/schema.graphqls`

Defines:

- Types and their fields
- Queries and mutations
- Input and output structures
- Relationships between types

### 2. Resolvers

Located in: `app/code/{Vendor}/{Module}/Model/Resolver/`

Implement: `Magento\Framework\GraphQl\Query\ResolverInterface`

Responsible for:

- Fetching data
- Business logic delegation
- Error handling
- Authorization checks

### 3. Data Providers

Located in: `app/code/{Vendor}/{Module}/Model/`

Provide:

- Reusable data fetching logic
- Service layer integration
- Data transformation

## Request Flow Diagram

```text
HTTP Request (/graphql)
    ↓
Query Parser
    ↓
Schema Validator
    ↓
Resolver Execution
    ↓
Data Provider
    ↓
Response Formatter
    ↓
JSON Response
```

## Module Structure Example

```text
app/code/Vendor/Module/
├── etc/
│   ├── module.xml
│   ├── schema.graphqls
│   └── di.xml
├── Model/
│   ├── Resolver/
│   │   ├── ProductList.php
│   │   └── ProductDetail.php
│   └── DataProvider/
│       └── ProductDataProvider.php
├── registration.php
└── composer.json
```

## Next Steps

Start with [Architecture Overview](01-overview.md) for a high-level understanding.

---

[← Previous: Getting Started](../02-getting-started/README.md) | [Back to Main](../../README.md) | [Next: Built-in Functionality →](../04-built-in-functionality/README.md)
