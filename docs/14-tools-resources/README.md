# Tools & Resources

## Overview

A comprehensive collection of tools, libraries, and resources for Magento 2 GraphQL development.

## Topics Covered

1. [Development Tools](01-development-tools.md)
2. [GraphQL Clients](02-graphql-clients.md)
3. [Testing Tools](03-testing-tools.md)
4. [Documentation Tools](04-documentation-tools.md)
5. [Performance Tools](05-performance-tools.md)
6. [Community Resources](06-community-resources.md)
7. [Learning Resources](07-learning-resources.md)
8. [Useful Extensions](08-extensions.md)

## GraphQL IDE Tools

### 1. GraphiQL (Built-in)

**Built into Magento** - No installation required

Access at: `https://your-store.com/graphql`

Features:

- Query editor with syntax highlighting
- Auto-completion
- Query history
- Schema documentation explorer
- Variable support

### 2. Altair GraphQL Client

**Desktop and Browser Extension**

Download: [https://altairgraphql.dev/](https://altairgraphql.dev/)

Features:

- Beautiful UI
- Query collections
- Pre-request scripts
- Multiple windows
- Import/export queries
- Authentication support

### 3. GraphQL Playground

**Standalone IDE**

Install:

```bash
npm install -g graphql-playground
```

Features:

- Interactive query editor
- Schema documentation
- Query history
- Multiple tabs
- Variables and headers support

### 4. Postman

**API Development Platform**

Download: [https://www.postman.com/](https://www.postman.com/)

Features:

- GraphQL support
- Collections
- Environment variables
- Automated testing
- Team collaboration
- CI/CD integration

### 5. Insomnia

**API Client**

Download: [https://insomnia.rest/](https://insomnia.rest/)

Features:

- GraphQL support
- Code generation
- Environment management
- Plugin system

## GraphQL Client Libraries

### 1. Apollo Client

**Most Popular React/JavaScript Client**

```bash
npm install @apollo/client graphql
```

```javascript
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';

const client = new ApolloClient({
  uri: 'https://your-store.com/graphql',
  cache: new InMemoryCache()
});

const GET_PRODUCTS = gql`
  query GetProducts {
    products(search: "shirt") {
      items {
        name
        sku
        price {
          regularPrice {
            amount {
              value
            }
          }
        }
      }
    }
  }
`;

client.query({ query: GET_PRODUCTS })
  .then(result => console.log(result));
```

### 2. URQL

**Lightweight GraphQL Client**

```bash
npm install urql graphql
```

### 3. Relay

**Facebook's GraphQL Client**

```bash
npm install react-relay relay-runtime
```

### 4. GraphQL Request

**Minimal GraphQL Client**

```bash
npm install graphql-request graphql
```

```javascript
import { request, gql } from 'graphql-request';

const query = gql`
  {
    storeConfig {
      store_name
      locale
    }
  }
`;

request('https://your-store.com/graphql', query)
  .then(data => console.log(data));
```

## Code Generation Tools

### 1. GraphQL Code Generator

**Automatic TypeScript/Types Generation**

```bash
npm install -D @graphql-codegen/cli
```

```yaml
# codegen.yml
schema: https://your-store.com/graphql
documents: './src/**/*.graphql'
generates:
  ./src/generated/graphql.ts:
    plugins:
      - typescript
      - typescript-operations
      - typescript-react-apollo
```

### 2. Apollo Codegen

```bash
npm install -g apollo
apollo client:codegen --target typescript
```

## Testing Tools

### 1. Jest with GraphQL

```bash
npm install --save-dev jest @types/jest
```

### 2. Newman (Postman CLI)

```bash
npm install -g newman
newman run magento-graphql-tests.json -e environment.json
```

### 3. Artillery (Load Testing)

```bash
npm install -g artillery
```

```yaml
# artillery-test.yml
config:
  target: 'https://your-store.com'
  phases:
    - duration: 60
      arrivalRate: 10
scenarios:
  - name: "GraphQL Product Query"
    flow:
      - post:
          url: "/graphql"
          json:
            query: "{ products(search: \"shirt\") { items { name sku } } }"
```

## Performance Tools

### 1. GraphQL Query Analyzer

Analyze query complexity and depth

### 2. Chrome DevTools

Monitor network requests and performance

### 3. New Relic

APM for GraphQL monitoring

### 4. Blackfire.io

PHP Performance profiling

## Browser Extensions

### 1. Apollo Client DevTools

Chrome/Firefox extension for debugging Apollo Client

### 2. GraphQL Network Inspector

View GraphQL requests in browser DevTools

## Documentation Tools

### 1. GraphQL Voyager

**Interactive Schema Visualization**

```bash
npx graphql-voyager
```

### 2. Spectaql

**Generate Static GraphQL Documentation**

```bash
npm install -g spectaql
```

### 3. GraphDoc

**Static Documentation Generator**

```bash
npm install -g @2fd/graphdoc
```

## Magento-Specific Tools

### 1. PWA Studio

**Official Magento PWA Toolkit**

```bash
npm create @magento/pwa
```

### 2. Vue Storefront

**PWA for eCommerce**

[https://www.vuestorefront.io/](https://www.vuestorefront.io/)

### 3. ScandiPWA

**Magento PWA Theme**

[https://scandipwa.com/](https://scandipwa.com/)

### 4. Magento GraphQL Playground Module

Enhanced GraphQL playground for Magento

## Official Documentation

### Magento Resources

- [Magento GraphQL Developer Guide](https://developer.adobe.com/commerce/webapi/graphql/)
- [PWA Studio Documentation](https://developer.adobe.com/commerce/pwa-studio/)
- [Magento DevDocs](https://devdocs.magento.com/)
- [Magento Community Forums](https://community.magento.com/)

### GraphQL Resources

- [GraphQL.org](https://graphql.org/)
- [How to GraphQL](https://www.howtographql.com/)
- [GraphQL Spec](https://spec.graphql.org/)
- [Apollo GraphQL Blog](https://www.apollographql.com/blog/)

## Community Resources

### GitHub Repositories

- [Magento GraphQL](https://github.com/magento/magento2/tree/2.4-develop/app/code/Magento/GraphQl)
- [PWA Studio](https://github.com/magento/pwa-studio)
- [Awesome GraphQL](https://github.com/chentsulin/awesome-graphql)

### Blogs and Tutorials

- [Magento Blog](https://business.adobe.com/blog/)
- [Magento Stack Exchange](https://magento.stackexchange.com/)
- [Magento Developers Blog](https://developer.adobe.com/commerce/blog/)

### YouTube Channels

- Adobe Commerce
- Magento Community Engineering
- Magento PWA Studio

### Podcasts

- Magento Community Podcast
- GraphQL Radio

## Useful PHP Packages

### 1. webonyx/graphql-php

Core GraphQL implementation for PHP

```bash
composer require webonyx/graphql-php
```

### 2. GraphQL Bundle (Symfony)

For Symfony-based projects

```bash
composer require overblog/graphql-bundle
```

## Configuration Files

### .editorconfig

```ini
[*.{graphql,gql}]
indent_style = space
indent_size = 2
```

### .graphqlconfig

```json
{
  "schema": "https://your-store.com/graphql",
  "documents": "**/*.graphql",
  "extensions": {
    "endpoints": {
      "dev": "https://dev.your-store.com/graphql",
      "prod": "https://your-store.com/graphql"
    }
  }
}
```

## VSCode Extensions

1. **GraphQL** by Prisma
2. **Apollo GraphQL** by Apollo GraphQL
3. **GraphQL for VSCode** by Kumar Harsh
4. **REST Client** by Huachao Mao (for testing)
5. **Prettier** with GraphQL support

## Quick Reference

### Common Query Patterns

```graphql
# Simple query
{ storeConfig { store_name } }

# With variables
query GetProduct($sku: String!) {
  products(filter: {sku: {eq: $sku}}) {
    items { name }
  }
}

# Mutation
mutation CreateCart {
  createEmptyCart
}

# Fragment
fragment ProductFields on ProductInterface {
  sku
  name
  price { regularPrice { amount { value } } }
}
```

### Useful Commands

```bash
# Test endpoint
curl -X POST https://your-store.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ storeConfig { store_name } }"}'

# With authentication
curl -X POST https://your-store.com/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"query":"{ customer { email } }"}'
```

## Learning Path

### Beginner

1. GraphQL.org tutorial
2. How to GraphQL course
3. Magento GraphQL basics
4. Simple query implementation

### Intermediate

1. Apollo Client documentation
2. Custom resolver development
3. Authentication implementation
4. Performance optimization

### Advanced

1. Schema design patterns
2. Complex resolver patterns
3. Caching strategies
4. Production deployment

## Cheat Sheets

- [GraphQL Cheat Sheet](https://devhints.io/graphql)
- [Apollo Client Cheat Sheet](https://www.apollographql.com/docs/react/api/core/ApolloClient/)
- [Magento GraphQL Reference](https://developer.adobe.com/commerce/webapi/graphql/schema/)

## Support

### Getting Help

- [Magento Stack Exchange](https://magento.stackexchange.com/)
- [Magento Community Forums](https://community.magento.com/)
- [GitHub Issues](https://github.com/magento/magento2/issues)
- [GraphQL Discord](https://discord.graphql.org/)

### Contributing

- Report bugs on GitHub
- Submit pull requests
- Contribute to documentation
- Share knowledge in forums

---

[← Previous: Examples](../13-examples/README.md) | [Back to Main](../../README.md)
