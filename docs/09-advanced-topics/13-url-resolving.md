# URL Resolving in Magento GraphQL

## Overview

URL resolving is a crucial feature in Magento GraphQL that allows you to resolve any URL (product, category, CMS page) to its corresponding entity. This is essential for headless storefronts and PWAs.

## The urlResolver Query

Magento provides the `urlResolver` query to identify and retrieve entities based on their URL key or path.

### Basic Syntax

```graphql
{
  urlResolver(url: String!): EntityUrl
}
```

### Return Type: EntityUrl

```graphql
type EntityUrl {
  id: Int
  canonical_url: String
  relative_url: String
  redirectCode: Int
  type: UrlRewriteEntityTypeEnum
}

enum UrlRewriteEntityTypeEnum {
  PRODUCT
  CATEGORY
  CMS_PAGE
}
```

## Use Cases

### 1. Resolve Product URL

```graphql
{
  urlResolver(url: "joust-duffle-bag.html") {
    id
    type
    canonical_url
    relative_url
  }
}
```

**Response:**
```json
{
  "data": {
    "urlResolver": {
      "id": 1,
      "type": "PRODUCT",
      "canonical_url": "https://example.com/joust-duffle-bag.html",
      "relative_url": "joust-duffle-bag.html"
    }
  }
}
```

### 2. Resolve Category URL

```graphql
{
  urlResolver(url: "men/tops-men.html") {
    id
    type
    canonical_url
    relative_url
  }
}
```

**Response:**
```json
{
  "data": {
    "urlResolver": {
      "id": 12,
      "type": "CATEGORY",
      "canonical_url": "https://example.com/men/tops-men.html",
      "relative_url": "men/tops-men.html"
    }
  }
}
```

### 3. Resolve CMS Page URL

```graphql
{
  urlResolver(url: "about-us") {
    id
    type
    canonical_url
    relative_url
  }
}
```

**Response:**
```json
{
  "data": {
    "urlResolver": {
      "id": 5,
      "type": "CMS_PAGE",
      "canonical_url": "https://example.com/about-us",
      "relative_url": "about-us"
    }
  }
}
```

## Complete Routing Solution

### Query with Entity Details

```graphql
query ResolveUrl($url: String!) {
  urlResolver(url: $url) {
    id
    type
    relative_url
    canonical_url
    redirectCode
  }
}
```

### Then Fetch Specific Entity Data

Based on the `type` returned, fetch the appropriate entity:

#### If type is PRODUCT:

```graphql
query GetProduct($id: Int!) {
  products(filter: { entity_id: { eq: $id } }) {
    items {
      id
      sku
      name
      url_key
      description {
        html
      }
      price_range {
        minimum_price {
          regular_price {
            value
            currency
          }
        }
      }
      media_gallery {
        url
        label
      }
    }
  }
}
```

#### If type is CATEGORY:

```graphql
query GetCategory($id: Int!) {
  category(id: $id) {
    id
    name
    url_key
    url_path
    description
    image
    product_count
    products {
      items {
        id
        name
        sku
      }
    }
  }
}
```

#### If type is CMS_PAGE:

```graphql
query GetCmsPage($id: Int!) {
  cmsPage(id: $id) {
    identifier
    url_key
    title
    content
    content_heading
    meta_title
    meta_description
  }
}
```

## URL Rewrites and Redirects

### Handling 301 Redirects

```graphql
{
  urlResolver(url: "old-product-url.html") {
    id
    type
    canonical_url
    relative_url
    redirectCode
  }
}
```

**Response with redirect:**
```json
{
  "data": {
    "urlResolver": {
      "id": 1,
      "type": "PRODUCT",
      "canonical_url": "https://example.com/new-product-url.html",
      "relative_url": "new-product-url.html",
      "redirectCode": 301
    }
  }
}
```

### Handling Redirect Codes

- **0**: No redirect (canonical URL)
- **301**: Permanent redirect
- **302**: Temporary redirect

## Implementation in Frontend

### React/Apollo Example

```javascript
import { gql, useQuery } from '@apollo/client';
import { useRouter } from 'next/router';

const RESOLVE_URL = gql`
  query ResolveUrl($url: String!) {
    urlResolver(url: $url) {
      id
      type
      canonical_url
      relative_url
      redirectCode
    }
  }
`;

const GET_PRODUCT = gql`
  query GetProduct($id: Int!) {
    products(filter: { entity_id: { eq: $id } }) {
      items {
        id
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
`;

const GET_CATEGORY = gql`
  query GetCategory($id: Int!) {
    category(id: $id) {
      id
      name
      url_path
      products {
        items {
          id
          name
        }
      }
    }
  }
`;

const GET_CMS_PAGE = gql`
  query GetCmsPage($id: Int!) {
    cmsPage(id: $id) {
      identifier
      title
      content
    }
  }
`;

function DynamicPage() {
  const router = useRouter();
  const { url } = router.query;

  // Step 1: Resolve URL
  const { data: urlData, loading: urlLoading } = useQuery(RESOLVE_URL, {
    variables: { url },
    skip: !url,
  });

  // Handle redirects
  useEffect(() => {
    if (urlData?.urlResolver?.redirectCode) {
      const { redirectCode, relative_url } = urlData.urlResolver;
      if (redirectCode === 301 || redirectCode === 302) {
        router.push(relative_url);
      }
    }
  }, [urlData]);

  // Step 2: Fetch entity based on type
  const entityId = urlData?.urlResolver?.id;
  const entityType = urlData?.urlResolver?.type;

  // Conditional queries based on type
  const { data: productData } = useQuery(GET_PRODUCT, {
    variables: { id: entityId },
    skip: entityType !== 'PRODUCT',
  });

  const { data: categoryData } = useQuery(GET_CATEGORY, {
    variables: { id: entityId },
    skip: entityType !== 'CATEGORY',
  });

  const { data: cmsData } = useQuery(GET_CMS_PAGE, {
    variables: { id: entityId },
    skip: entityType !== 'CMS_PAGE',
  });

  if (urlLoading) return <div>Loading...</div>;

  // Render based on entity type
  switch (entityType) {
    case 'PRODUCT':
      return <ProductPage data={productData} />;
    case 'CATEGORY':
      return <CategoryPage data={categoryData} />;
    case 'CMS_PAGE':
      return <CmsPage data={cmsData} />;
    default:
      return <NotFound />;
  }
}
```

### Vue Example

```vue
<template>
  <div>
    <ProductPage v-if="entityType === 'PRODUCT'" :data="entityData" />
    <CategoryPage v-else-if="entityType === 'CATEGORY'" :data="entityData" />
    <CmsPage v-else-if="entityType === 'CMS_PAGE'" :data="entityData" />
    <NotFound v-else />
  </div>
</template>

<script>
import { useQuery } from '@vue/apollo-composable';
import { computed, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import gql from 'graphql-tag';

export default {
  setup() {
    const route = useRoute();
    const router = useRouter();
    const url = computed(() => route.params.url);

    // Resolve URL
    const { result: urlResult } = useQuery(gql`
      query ResolveUrl($url: String!) {
        urlResolver(url: $url) {
          id
          type
          canonical_url
          redirectCode
        }
      }
    `, { url });

    // Handle redirects
    watch(() => urlResult.value?.urlResolver, (resolver) => {
      if (resolver?.redirectCode) {
        router.push(resolver.canonical_url);
      }
    });

    const entityType = computed(() => urlResult.value?.urlResolver?.type);
    const entityId = computed(() => urlResult.value?.urlResolver?.id);

    // Fetch entity data based on type
    const { result: entityData } = useQuery(
      computed(() => getQueryForType(entityType.value)),
      { id: entityId }
    );

    return {
      entityType,
      entityData,
    };
  },
};
</script>
```

## Advanced URL Routing

### Multi-Store URL Resolution

```graphql
query ResolveUrlWithStore($url: String!) {
  urlResolver(url: $url) {
    id
    type
    canonical_url
    relative_url
  }
}
```

**Headers:**
```http
Store: default
Content-Type: application/json
```

### Breadcrumb Generation

```graphql
query GetCategoryBreadcrumbs($id: Int!) {
  category(id: $id) {
    id
    name
    url_path
    breadcrumbs {
      category_id
      category_name
      category_url_path
    }
  }
}
```

### SEO Canonical URLs

```graphql
query GetProductCanonical($url: String!) {
  urlResolver(url: $url) {
    canonical_url
    relative_url
  }
}
```

Use the canonical_url for `<link rel="canonical">` tag:

```html
<link rel="canonical" href="{{ canonicalUrl }}" />
```

## URL Generation

### Generate Product URL

```graphql
{
  products(filter: { sku: { eq: "24-MB01" } }) {
    items {
      name
      url_key
      url_suffix
    }
  }
}
```

Construct URL:
```javascript
const productUrl = `${product.url_key}${product.url_suffix || '.html'}`;
```

### Generate Category URL

```graphql
{
  category(id: 3) {
    name
    url_path
    url_suffix
  }
}
```

Construct URL:
```javascript
const categoryUrl = `${category.url_path}${category.url_suffix || ''}`;
```

## Custom URL Resolver Implementation

### Creating a Custom URL Resolver

```php
<?php
// etc/schema.graphqls
type Query {
    customUrlResolver(url: String!): CustomEntityUrl
        @resolver(class: "Vendor\\Module\\Model\\Resolver\\CustomUrlResolver")
}

type CustomEntityUrl {
    entity_id: Int
    entity_type: String
    url_key: String
    store_id: Int
}
```

```php
<?php
namespace Vendor\Module\Model\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Query\ResolverInterface;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;
use Magento\UrlRewrite\Model\UrlFinderInterface;

class CustomUrlResolver implements ResolverInterface
{
    private $urlFinder;

    public function __construct(UrlFinderInterface $urlFinder)
    {
        $this->urlFinder = $urlFinder;
    }

    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) {
        $url = $args['url'];
        $storeId = $context->getExtensionAttributes()->getStore()->getId();

        $urlRewrite = $this->urlFinder->findOneByData([
            'request_path' => $url,
            'store_id' => $storeId
        ]);

        if (!$urlRewrite) {
            return null;
        }

        return [
            'entity_id' => $urlRewrite->getEntityId(),
            'entity_type' => $urlRewrite->getEntityType(),
            'url_key' => $urlRewrite->getRequestPath(),
            'store_id' => $urlRewrite->getStoreId()
        ];
    }
}
```

## Best Practices

### 1. Cache URL Resolution Results

```javascript
// Apollo Client with cache
const client = new ApolloClient({
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          urlResolver: {
            keyArgs: ['url'],
            merge(existing, incoming) {
              return incoming;
            },
          },
        },
      },
    },
  }),
});
```

### 2. Handle 404 Pages

```javascript
const { data, error } = useQuery(RESOLVE_URL, {
  variables: { url },
});

if (!data?.urlResolver || error) {
  return <NotFoundPage />;
}
```

### 3. Implement Client-Side Routing

```javascript
// Next.js catch-all route
// pages/[...url].js
export default function DynamicPage({ urlData, entityData }) {
  // Render based on entity type
}

export async function getServerSideProps({ params }) {
  const url = params.url.join('/');

  // Resolve URL
  const urlData = await resolveUrl(url);

  if (urlData.redirectCode) {
    return {
      redirect: {
        destination: urlData.relative_url,
        permanent: urlData.redirectCode === 301,
      },
    };
  }

  // Fetch entity data
  const entityData = await fetchEntity(urlData.type, urlData.id);

  return {
    props: {
      urlData,
      entityData,
    },
  };
}
```

### 4. Optimize for SEO

```javascript
// Generate metadata based on entity type
export async function generateMetadata({ params }) {
  const url = params.url.join('/');
  const urlData = await resolveUrl(url);
  const entityData = await fetchEntity(urlData.type, urlData.id);

  return {
    title: entityData.meta_title || entityData.name,
    description: entityData.meta_description,
    canonical: urlData.canonical_url,
    openGraph: {
      url: urlData.canonical_url,
      type: urlData.type.toLowerCase(),
    },
  };
}
```

## Troubleshooting

### URL Not Resolving

```graphql
# Check URL rewrite table
{
  urlResolver(url: "test-product.html") {
    id
    type
  }
}

# If returns null, check:
# 1. URL key is correct
# 2. Product/Category is enabled
# 3. URL rewrites are generated
# 4. Store view is correct
```

### Redirect Loop

```javascript
// Prevent infinite redirects
const [redirectCount, setRedirectCount] = useState(0);

useEffect(() => {
  if (urlData?.urlResolver?.redirectCode && redirectCount < 3) {
    setRedirectCount(prev => prev + 1);
    router.push(urlData.urlResolver.relative_url);
  }
}, [urlData]);
```

---

[← Previous: Extending Core Types](12-extending-core.md) | [Back to Advanced Topics](README.md) | [Next: Security →](../11-security/README.md)
