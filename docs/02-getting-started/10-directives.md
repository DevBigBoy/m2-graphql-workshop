# GraphQL Directives

## Overview

Directives are special annotations in GraphQL that allow you to modify the execution behavior of queries and mutations. They are prefixed with `@` symbol.

## What are Directives?

Directives provide a way to dynamically change the structure and shape of queries using variables. They allow conditional inclusion or exclusion of fields based on runtime values.

## Built-in GraphQL Directives

### 1. @include Directive

Include a field based on a boolean condition.

**Syntax:**
```graphql
@include(if: Boolean!)
```

**Example:**

```graphql
query GetProduct($sku: String!, $includeDescription: Boolean!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      id
      sku
      name
      description @include(if: $includeDescription) {
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
    }
  }
}
```

**Variables:**
```json
{
  "sku": "24-MB01",
  "includeDescription": true
}
```

**Use Case in Magento:**
```graphql
query GetProducts($showReviews: Boolean!, $showRelated: Boolean!) {
  products(filter: { category_id: { eq: "3" } }) {
    items {
      name
      sku
      reviews @include(if: $showReviews) {
        items {
          summary
          text
        }
      }
      related_products @include(if: $showRelated) {
        name
        sku
      }
    }
  }
}
```

### 2. @skip Directive

Skip a field based on a boolean condition (opposite of @include).

**Syntax:**
```graphql
@skip(if: Boolean!)
```

**Example:**

```graphql
query GetProduct($sku: String!, $skipImages: Boolean!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      id
      sku
      name
      media_gallery @skip(if: $skipImages) {
        url
        label
        position
      }
      image @skip(if: $skipImages) {
        url
      }
    }
  }
}
```

**Variables:**
```json
{
  "sku": "24-MB01",
  "skipImages": false
}
```

**Use Case - Performance Optimization:**
```graphql
query GetProducts($isMobile: Boolean!) {
  products(search: "shirt") {
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
      # Skip heavy fields on mobile
      media_gallery @skip(if: $isMobile) {
        url
        label
      }
      description @skip(if: $isMobile) {
        html
      }
      related_products @skip(if: $isMobile) {
        name
      }
    }
  }
}
```

### 3. @deprecated Directive

Marks fields as deprecated in the schema (used in schema definition, not queries).

**Schema Definition:**
```graphql
type Product {
  id: Int
  name: String
  old_price: Float @deprecated(reason: "Use price_range instead")
  price_range: PriceRange
}
```

## Combining @include and @skip

You can use both directives on the same field:

```graphql
query GetCustomer($includeOrders: Boolean!, $skipAddresses: Boolean!) {
  customer {
    firstname
    lastname
    email
    orders @include(if: $includeOrders) {
      items {
        order_number
        order_date
        total {
          grand_total {
            value
          }
        }
      }
    }
    addresses @skip(if: $skipAddresses) {
      street
      city
      postcode
    }
  }
}
```

## Magento-Specific Use Cases

### 1. Conditional Product Information

```graphql
query GetProductDetails(
  $sku: String!
  $includeReviews: Boolean!
  $includeRelated: Boolean!
  $includeUpsell: Boolean!
) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name
      sku
      description {
        html
      }

      # Only fetch reviews if needed
      reviews @include(if: $includeReviews) {
        items {
          average_rating
          summary
          text
          nickname
        }
      }

      # Only fetch related products if needed
      related_products @include(if: $includeRelated) {
        sku
        name
        price_range {
          minimum_price {
            regular_price {
              value
            }
          }
        }
      }

      # Only fetch upsell products if needed
      upsell_products @include(if: $includeUpsell) {
        sku
        name
      }
    }
  }
}
```

### 2. Customer Data Privacy

```graphql
query GetCustomer($showSensitiveData: Boolean!) {
  customer {
    firstname
    lastname

    # Only include sensitive data when authorized
    email @include(if: $showSensitiveData)
    date_of_birth @include(if: $showSensitiveData)
    taxvat @include(if: $showSensitiveData)

    addresses @include(if: $showSensitiveData) {
      street
      city
      telephone
    }
  }
}
```

### 3. Feature Flags

```graphql
query GetCart(
  $cartId: String!
  $rewardPointsEnabled: Boolean!
  $giftWrapEnabled: Boolean!
) {
  cart(cart_id: $cartId) {
    id
    items {
      id
      product {
        name
        sku
      }
      quantity
    }

    # Only show if feature is enabled
    applied_reward_points @include(if: $rewardPointsEnabled) {
      points
      money {
        value
      }
    }

    # Only show if feature is enabled
    gift_wrapping @include(if: $giftWrapEnabled) {
      id
      design
      price {
        value
      }
    }
  }
}
```

### 4. Responsive Design

```graphql
query GetProductListing(
  $categoryId: String!
  $isDesktop: Boolean!
  $isMobile: Boolean!
) {
  products(filter: { category_id: { eq: $categoryId } }) {
    items {
      name
      sku

      # Full image for desktop
      image @include(if: $isDesktop) {
        url
      }

      # Small image for mobile
      small_image @include(if: $isMobile) {
        url
      }

      # Full description for desktop only
      description @include(if: $isDesktop) {
        html
      }

      # Short description for mobile
      short_description @include(if: $isMobile) {
        html
      }
    }
  }
}
```

### 5. A/B Testing

```graphql
query GetProducts(
  $showVariantA: Boolean!
  $showVariantB: Boolean!
) {
  products(search: "shirt") {
    items {
      name
      sku

      # Variant A: Show regular price prominently
      price_range @include(if: $showVariantA) {
        minimum_price {
          regular_price {
            value
            currency
          }
        }
      }

      # Variant B: Show discount information prominently
      price_range @include(if: $showVariantB) {
        minimum_price {
          regular_price {
            value
          }
          discount {
            amount_off
            percent_off
          }
        }
      }
    }
  }
}
```

## Custom Directives in Magento

While GraphQL allows defining custom directives, Magento 2 primarily uses the standard directives. However, you can create custom directives for specific needs.

### Example: Custom @cache Directive (Conceptual)

```graphql
# Schema definition
directive @cache(ttl: Int) on FIELD_DEFINITION

type Query {
  products(filter: ProductFilterInput): Products @cache(ttl: 3600)
  storeConfig: StoreConfig @cache(ttl: 86400)
}
```

## Best Practices

### 1. Use Directives for Conditional Logic

```graphql
# Good - Dynamic field inclusion
query GetProduct($sku: String!, $detailed: Boolean!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name
      sku
      description @include(if: $detailed) {
        html
      }
      media_gallery @include(if: $detailed) {
        url
      }
    }
  }
}

# Bad - Multiple separate queries
query GetProductBasic($sku: String!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name
      sku
    }
  }
}

query GetProductDetailed($sku: String!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name
      sku
      description {
        html
      }
      media_gallery {
        url
      }
    }
  }
}
```

### 2. Optimize for Performance

```graphql
query GetProducts(
  $pageSize: Int!
  $currentPage: Int!
  $loadHeavyData: Boolean!
) {
  products(
    pageSize: $pageSize
    currentPage: $currentPage
  ) {
    items {
      name
      sku

      # Only load heavy fields when needed
      description @include(if: $loadHeavyData) {
        html
      }
      media_gallery @include(if: $loadHeavyData) {
        url
        label
      }
      related_products @include(if: $loadHeavyData) {
        name
      }
    }
  }
}
```

### 3. Use with Variables

```graphql
query GetCustomer(
  $includeOrders: Boolean! = false
  $includeWishlist: Boolean! = false
) {
  customer {
    firstname
    lastname
    email
    orders @include(if: $includeOrders) {
      items {
        order_number
      }
    }
    wishlist @include(if: $includeWishlist) {
      items {
        product {
          name
        }
      }
    }
  }
}
```

## Common Patterns

### Pattern 1: Loading States

```graphql
query GetData(
  $quick: Boolean!
  $detailed: Boolean!
) {
  products {
    items {
      # Always load
      name
      sku

      # Quick load
      price_range @include(if: $quick) {
        minimum_price {
          regular_price {
            value
          }
        }
      }

      # Detailed load
      description @include(if: $detailed) {
        html
      }
      media_gallery @include(if: $detailed) {
        url
      }
    }
  }
}
```

### Pattern 2: User Permissions

```graphql
query GetProduct(
  $sku: String!
  $isAdmin: Boolean!
  $isCustomer: Boolean!
) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name
      sku

      # Admin only
      cost @include(if: $isAdmin)
      stock_status @include(if: $isAdmin)

      # Customer only
      price_range @include(if: $isCustomer) {
        minimum_price {
          final_price {
            value
          }
        }
      }
    }
  }
}
```

### Pattern 3: Internationalization

```graphql
query GetProduct(
  $sku: String!
  $showTranslations: Boolean!
) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name

      description @include(if: $showTranslations) {
        html
      }

      meta_title @include(if: $showTranslations)
      meta_description @include(if: $showTranslations)
    }
  }
}
```

## Performance Impact

### Query Size Optimization

```javascript
// JavaScript example with Apollo Client
const GET_PRODUCT = gql`
  query GetProduct(
    $sku: String!
    $includeReviews: Boolean!
    $includeRelated: Boolean!
  ) {
    products(filter: { sku: { eq: $sku } }) {
      items {
        name
        sku
        reviews @include(if: $includeReviews) {
          items {
            summary
          }
        }
        related_products @include(if: $includeRelated) {
          name
        }
      }
    }
  }
`;

// Usage
const { data } = useQuery(GET_PRODUCT, {
  variables: {
    sku: '24-MB01',
    includeReviews: true,  // Only when needed
    includeRelated: false  // Skip heavy data
  }
});
```

## Troubleshooting

### Directive Not Working

```graphql
# Wrong - Missing exclamation mark
query GetProduct($includeDesc: Boolean) {
  products {
    items {
      description @include(if: $includeDesc) {
        html
      }
    }
  }
}

# Correct - Boolean! is required
query GetProduct($includeDesc: Boolean!) {
  products {
    items {
      description @include(if: $includeDesc) {
        html
      }
    }
  }
}
```

### Variable Not Provided

```json
// Error: Variable includeDesc not provided

// Solution: Always provide the variable
{
  "includeDesc": true
}
```

---

[← Previous: Fragments](09-fragments.md) | [Back to Getting Started](README.md) | [Next: Architecture →](../03-architecture/README.md)
