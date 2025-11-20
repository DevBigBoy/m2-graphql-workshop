# Tips & Tricks for Magento 2 GraphQL

## Overview

A collection of practical tips, tricks, and shortcuts to make your Magento GraphQL development more efficient and effective.

## Quick Tips

### 1. Use Aliases for Multiple Queries

Fetch the same field with different parameters in a single request:

```graphql
{
  shirts: products(filter: { category_id: { eq: "3" } }) {
    items {
      name
      sku
    }
  }

  pants: products(filter: { category_id: { eq: "4" } }) {
    items {
      name
      sku
    }
  }

  featured: products(filter: { category_id: { eq: "5" } }) {
    items {
      name
      sku
    }
  }
}
```

### 2. Use Variables for Reusable Queries

```graphql
query GetProducts($categoryId: String!, $pageSize: Int = 20) {
  products(
    filter: { category_id: { eq: $categoryId } }
    pageSize: $pageSize
  ) {
    items {
      name
      sku
    }
  }
}
```

### 3. Introspection for Schema Discovery

```graphql
# Get all available types
{
  __schema {
    types {
      name
      kind
    }
  }
}

# Get details about a specific type
{
  __type(name: "ProductInterface") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

### 4. Use Fragments for Repeated Fields

```graphql
fragment ProductCard on ProductInterface {
  id
  sku
  name
  price_range {
    minimum_price {
      regular_price {
        value
        currency
      }
    }
  }
  small_image {
    url
  }
}

query GetMultipleProducts {
  newArrivals: products(filter: { new: { eq: "1" } }) {
    items {
      ...ProductCard
    }
  }

  featured: products(filter: { featured: { eq: "1" } }) {
    items {
      ...ProductCard
    }
  }
}
```

## Performance Tips

### 5. Request Only Needed Fields

```graphql
# Bad - Over-fetching
{
  products {
    items {
      id
      sku
      name
      description { html }
      short_description { html }
      meta_title
      meta_description
      media_gallery { url label }
      related_products { name }
      # ... many more fields
    }
  }
}

# Good - Specific fields
{
  products {
    items {
      name
      sku
      price_range {
        minimum_price {
          regular_price {
            value
          }
        }
      }
    }
  }
}
```

### 6. Use Pagination

```graphql
query GetProducts($page: Int = 1, $pageSize: Int = 20) {
  products(
    currentPage: $page
    pageSize: $pageSize
  ) {
    items {
      name
      sku
    }
    page_info {
      current_page
      page_size
      total_pages
    }
    total_count
  }
}
```

### 7. Batch Related Queries

```graphql
# Instead of multiple separate requests
{
  customer {
    firstname
    lastname
    email
  }

  cart(cart_id: "...") {
    items {
      product {
        name
      }
    }
  }

  storeConfig {
    store_name
  }
}
```

## Development Tips

### 8. Use GraphQL Playground for Testing

Access at: `https://your-magento-store.com/graphql`

Features:
- Syntax highlighting
- Auto-completion
- Schema documentation
- Query history

### 9. Enable Developer Mode for Better Errors

```bash
bin/magento deploy:mode:set developer
```

### 10. Log GraphQL Queries

```xml
<!-- etc/di.xml -->
<type name="Magento\GraphQl\Controller\GraphQl">
    <plugin name="log_graphql_queries" type="Vendor\Module\Plugin\LogQueries"/>
</type>
```

```php
<?php
namespace Vendor\Module\Plugin;

class LogQueries
{
    private $logger;

    public function __construct(\Psr\Log\LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function beforeDispatch($subject, $request)
    {
        $query = json_decode($request->getContent(), true);
        $this->logger->debug('GraphQL Query', [
            'query' => $query['query'] ?? '',
            'variables' => $query['variables'] ?? []
        ]);
    }
}
```

## Authentication Tips

### 11. Store Token Securely

```javascript
// Good - Use httpOnly cookies
document.cookie = `token=${token}; Secure; HttpOnly; SameSite=Strict`;

// Or use secure storage
localStorage.setItem('customerToken', token);

// Bad - Expose in URL
// https://example.com?token=xxx (Never do this!)
```

### 12. Check Token Expiration

```graphql
mutation {
  generateCustomerToken(
    email: "customer@example.com"
    password: "password"
  ) {
    token
  }
}
```

Set expiration reminder:
```javascript
const tokenExpiration = Date.now() + (3600 * 1000); // 1 hour
localStorage.setItem('tokenExpiration', tokenExpiration);
```

### 13. Refresh Token Pattern

```javascript
async function executeQuery(query, variables) {
  let token = localStorage.getItem('customerToken');
  const expiration = localStorage.getItem('tokenExpiration');

  // Check if token is about to expire
  if (Date.now() > expiration - (5 * 60 * 1000)) {
    token = await refreshToken();
  }

  return fetch('/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify({ query, variables })
  });
}
```

## Query Optimization Tips

### 14. Use Field Selection in Resolvers

```php
<?php
public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
{
    $fieldSelection = $info->getFieldSelection(5);

    // Only load description if requested
    if (isset($fieldSelection['description'])) {
        $product->getDescription();
    }

    // Only load media gallery if requested
    if (isset($fieldSelection['media_gallery'])) {
        $product->getMediaGallery();
    }
}
```

### 15. Implement DataLoader Pattern

```php
<?php
namespace Vendor\Module\Model\Resolver;

class ProductBatch implements ResolverInterface
{
    private $batchedIds = [];
    private $loader;

    public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
    {
        $productId = $value['product_id'];

        // Add to batch
        $this->batchedIds[] = $productId;

        // Return promise
        return function() use ($productId) {
            // Load all batched products at once
            $products = $this->loadProducts($this->batchedIds);
            return $products[$productId];
        };
    }

    private function loadProducts(array $ids)
    {
        $searchCriteria = $this->searchCriteriaBuilder
            ->addFilter('entity_id', $ids, 'in')
            ->create();

        return $this->productRepository->getList($searchCriteria);
    }
}
```

### 16. Cache Resolver Results

```php
<?php
public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
{
    $cacheKey = 'graphql_product_' . $args['sku'];

    // Check cache
    if ($cached = $this->cache->load($cacheKey)) {
        return json_decode($cached, true);
    }

    // Load data
    $product = $this->productRepository->get($args['sku']);
    $result = $this->formatProduct($product);

    // Save to cache
    $this->cache->save(
        json_encode($result),
        $cacheKey,
        [\Magento\Catalog\Model\Product::CACHE_TAG],
        3600
    );

    return $result;
}
```

## Error Handling Tips

### 17. Use Specific Exception Types

```php
<?php
use Magento\Framework\GraphQl\Exception\GraphQlInputException;
use Magento\Framework\GraphQl\Exception\GraphQlAuthorizationException;
use Magento\Framework\GraphQl\Exception\GraphQlNoSuchEntityException;

// Input validation
if (!$this->validator->isValid($input)) {
    throw new GraphQlInputException(__('Invalid input'));
}

// Not found
if (!$product) {
    throw new GraphQlNoSuchEntityException(__('Product not found'));
}

// Not authorized
if (!$this->isAuthorized($context)) {
    throw new GraphQlAuthorizationException(__('Not authorized'));
}
```

### 18. Client-Side Error Handling

```javascript
try {
  const { data } = await client.query({
    query: GET_PRODUCT,
    variables: { sku: '24-MB01' }
  });
} catch (error) {
  if (error.graphQLErrors) {
    error.graphQLErrors.forEach(err => {
      console.error('GraphQL Error:', err.message);

      // Handle specific error types
      if (err.extensions?.category === 'graphql-no-such-entity') {
        // Show 404 page
      } else if (err.extensions?.category === 'graphql-authorization') {
        // Redirect to login
      }
    });
  }

  if (error.networkError) {
    console.error('Network Error:', error.networkError);
  }
}
```

## Testing Tips

### 19. Use GraphQL Mocking

```javascript
import { MockedProvider } from '@apollo/client/testing';

const mocks = [
  {
    request: {
      query: GET_PRODUCT,
      variables: { sku: '24-MB01' }
    },
    result: {
      data: {
        products: {
          items: [{
            name: 'Test Product',
            sku: '24-MB01',
            price: { value: 50 }
          }]
        }
      }
    }
  }
];

<MockedProvider mocks={mocks} addTypename={false}>
  <ProductComponent sku="24-MB01" />
</MockedProvider>
```

### 20. Integration Test Shortcuts

```php
<?php
namespace Vendor\Module\Test\Integration;

class ProductQueryTest extends \Magento\TestFramework\TestCase\GraphQlAbstract
{
    public function testGetProduct()
    {
        $query = <<<QUERY
{
    products(filter: {sku: {eq: "simple"}}) {
        items {
            name
            sku
        }
    }
}
QUERY;

        $response = $this->graphQlQuery($query);

        $this->assertArrayHasKey('products', $response);
        $this->assertNotEmpty($response['products']['items']);
    }
}
```

## Debugging Tips

### 21. Add Query Complexity Monitoring

```php
<?php
namespace Vendor\Module\Plugin;

class QueryComplexityMonitor
{
    private $logger;

    public function beforeDispatch($subject, $request)
    {
        $query = json_decode($request->getContent(), true);
        $complexity = $this->calculateComplexity($query['query']);

        if ($complexity > 100) {
            $this->logger->warning('High query complexity', [
                'complexity' => $complexity,
                'query' => $query['query']
            ]);
        }
    }
}
```

### 22. Use Chrome DevTools Network Tab

Filter by:
- `/graphql` endpoint
- Check request payload
- Inspect response
- Monitor timing

### 23. Enable SQL Query Logging

```php
<!-- app/etc/env.php -->
'db' => [
    'connection' => [
        'default' => [
            'profiler' => [
                'class' => '\\Magento\\Framework\\DB\\Profiler',
                'enabled' => true
            ]
        ]
    ]
]
```

## Security Tips

### 24. Implement Rate Limiting

```php
<?php
namespace Vendor\Module\Plugin;

class RateLimiter
{
    private const RATE_LIMIT = 100;
    private const PERIOD = 60;

    public function beforeDispatch($subject, $request)
    {
        $ip = $request->getClientIp();
        $cacheKey = 'rate_limit_' . $ip;

        $count = $this->cache->load($cacheKey) ?: 0;

        if ($count >= self::RATE_LIMIT) {
            throw new \Exception('Rate limit exceeded');
        }

        $this->cache->save($count + 1, $cacheKey, [], self::PERIOD);
    }
}
```

### 25. Validate All Inputs

```php
<?php
public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
{
    // Validate required fields
    if (!isset($args['email']) || !filter_var($args['email'], FILTER_VALIDATE_EMAIL)) {
        throw new GraphQlInputException(__('Invalid email'));
    }

    // Sanitize inputs
    $email = filter_var($args['email'], FILTER_SANITIZE_EMAIL);
    $name = strip_tags($args['name']);

    // Validate length
    if (strlen($name) > 255) {
        throw new GraphQlInputException(__('Name too long'));
    }
}
```

### 26. Use HTTPS Only

```nginx
# nginx configuration
server {
    listen 443 ssl http2;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # Redirect HTTP to HTTPS
    if ($scheme = http) {
        return 301 https://$server_name$request_uri;
    }
}
```

## Frontend Integration Tips

### 27. Apollo Client Setup

```javascript
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: 'https://your-magento-store.com/graphql',
});

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('customerToken');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
      store: 'default',
    }
  };
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          products: {
            keyArgs: ['filter', 'sort'],
            merge(existing, incoming, { args }) {
              // Implement pagination merge logic
              if (args?.currentPage === 1) {
                return incoming;
              }
              return {
                ...incoming,
                items: [...(existing?.items || []), ...incoming.items]
              };
            }
          }
        }
      }
    }
  })
});
```

### 28. Custom Hooks for Common Operations

```javascript
// useProduct.js
import { useQuery, gql } from '@apollo/client';

const GET_PRODUCT = gql`
  query GetProduct($sku: String!) {
    products(filter: { sku: { eq: $sku } }) {
      items {
        id
        sku
        name
        price_range {
          minimum_price {
            regular_price {
              value
              currency
            }
          }
        }
      }
    }
  }
`;

export function useProduct(sku) {
  const { data, loading, error } = useQuery(GET_PRODUCT, {
    variables: { sku },
    skip: !sku
  });

  return {
    product: data?.products?.items?.[0],
    loading,
    error
  };
}

// Usage
function ProductComponent({ sku }) {
  const { product, loading, error } = useProduct(sku);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error!</div>;

  return <div>{product.name}</div>;
}
```

### 29. Optimistic UI Updates

```javascript
const [addToCart] = useMutation(ADD_TO_CART, {
  optimisticResponse: {
    addSimpleProductsToCart: {
      cart: {
        __typename: 'Cart',
        total_quantity: currentCart.total_quantity + quantity,
      }
    }
  },
  update(cache, { data }) {
    // Update cache immediately
  }
});
```

### 30. Error Boundary for GraphQL

```javascript
class GraphQLErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    if (error.graphQLErrors) {
      // Log GraphQL errors
      console.error('GraphQL Errors:', error.graphQLErrors);
    }
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}
```

## Bonus Tips

### 31. Use Query Naming

```graphql
# Good - Named query
query GetProductDetails($sku: String!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name
    }
  }
}

# Bad - Anonymous query
{
  products(filter: { sku: { eq: "24-MB01" } }) {
    items {
      name
    }
  }
}
```

### 32. Leverage Store Header

```http
POST /graphql HTTP/1.1
Host: your-magento-store.com
Content-Type: application/json
Store: us_en

{
  "query": "{ storeConfig { store_name } }"
}
```

### 33. Use Multi-part Requests for File Uploads

```javascript
const formData = new FormData();
formData.append('operations', JSON.stringify({
  query: 'mutation ($file: Upload!) { uploadFile(file: $file) }',
  variables: { file: null }
}));
formData.append('map', JSON.stringify({ '0': ['variables.file'] }));
formData.append('0', fileInput.files[0]);

fetch('/graphql', {
  method: 'POST',
  body: formData
});
```

---

[Back to Main](../README.md)
