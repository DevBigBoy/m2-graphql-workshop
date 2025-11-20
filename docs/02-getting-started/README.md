# Getting Started with Magento 2 GraphQL

## Overview

This section guides you through setting up your development environment and making your first GraphQL queries.

## Topics Covered

1. [Prerequisites](01-prerequisites.md)
2. [Development Environment Setup](02-environment-setup.md)
3. [GraphQL Endpoint](03-graphql-endpoint.md)
4. [GraphQL IDE Tools](04-ide-tools.md)
   - GraphiQL
   - Altair GraphQL Client
   - Postman
   - GraphQL Playground
5. [Your First Query](05-first-query.md)
6. [Introspection Queries](06-introspection.md)
7. [GraphQL Query Structure](07-query-structure.md)
8. [Using Variables](08-using-variables.md)
9. [Query Fragments](09-fragments.md)

## Learning Objectives

By the end of this section, you will be able to:

- Set up a development environment for Magento GraphQL
- Access the GraphQL endpoint
- Use GraphQL IDE tools
- Write and execute basic queries
- Use introspection to explore the schema
- Understand query structure and syntax
- Use variables and fragments

## Prerequisites

Before starting, ensure you have:

- Magento 2.3.0 or higher installed
- Admin access to Magento
- Basic understanding of HTTP and APIs
- A GraphQL client tool (GraphiQL, Postman, etc.)

## GraphQL Endpoint

Magento's GraphQL endpoint is available at:

```text
https://your-magento-domain.com/graphql
```

The endpoint accepts:

- **POST** requests for queries and mutations
- **GET** requests for queries only (with URL parameters)

## Quick Start Example

Here's a simple query to test your setup:

```graphql
{
  storeConfig {
    store_name
    store_code
    locale
    base_currency_code
  }
}
```

## Next Steps

Proceed to [Prerequisites](01-prerequisites.md) to ensure your environment is ready.

---

[← Previous: Introduction](../01-introduction/README.md) | [Back to Main](../../README.md) | [Next: Architecture →](../03-architecture/README.md)
