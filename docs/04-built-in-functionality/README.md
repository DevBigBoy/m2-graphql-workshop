# Built-in GraphQL Functionality

## Overview

Magento 2 comes with comprehensive GraphQL coverage for most e-commerce operations. This section explores all available queries and mutations.

## Topics Covered

1. [Catalog Operations](01-catalog/README.md)
   - Product queries
   - Category queries
   - Product search and filtering
   - Product types (simple, configurable, grouped, bundle)
   - Custom attributes
   - Reviews and ratings
2. [Customer Operations](02-customer/README.md)
   - Customer registration
   - Customer authentication
   - Profile management
   - Address management
   - Order history
   - Wishlist operations
3. [Cart & Checkout](03-cart-checkout/README.md)
   - Cart creation and management
   - Adding products to cart
   - Shipping methods
   - Payment methods
   - Order placement
   - Guest checkout
4. [CMS & Store Config](04-cms-store/README.md)
   - CMS pages and blocks
   - Store configuration
   - Currency
   - Countries
   - Available stores
5. [Additional Features](05-additional/README.md)
   - Compare products
   - Recently viewed
   - Gift cards
   - Reward points
   - Store credit

## Learning Objectives

By the end of this section, you will:

- Know all available built-in queries and mutations
- Understand how to query different product types
- Be able to implement cart and checkout flows
- Know how to fetch CMS content
- Understand customer authentication flows

## Most Common Queries

### Get Products

```graphql
{
  products(
    filter: { sku: { eq: "24-MB01" } }
  ) {
    items {
      sku
      name
      price {
        regularPrice {
          amount {
            value
            currency
          }
        }
      }
    }
  }
}
```

### Get Categories

```graphql
{
  categoryList {
    id
    name
    url_path
    children {
      id
      name
    }
  }
}
```

### Customer Token

```graphql
mutation {
  generateCustomerToken(
    email: "customer@example.com"
    password: "password123"
  ) {
    token
  }
}
```

### Create Cart

```graphql
mutation {
  createEmptyCart
}
```

## Query Examples by Use Case

- **Product Listing Page**: See [Catalog - Product Listing](01-catalog/product-listing.md)
- **Product Detail Page**: See [Catalog - Product Detail](01-catalog/product-detail.md)
- **Category Page**: See [Catalog - Categories](01-catalog/categories.md)
- **Login/Register**: See [Customer - Authentication](02-customer/authentication.md)
- **Add to Cart**: See [Cart - Add Products](03-cart-checkout/add-to-cart.md)
- **Checkout**: See [Cart - Checkout Flow](03-cart-checkout/checkout-flow.md)

## Next Steps

Explore each section to learn about specific functionality, starting with [Catalog Operations](01-catalog/README.md).

---

[← Previous: Architecture](../03-architecture/README.md) | [Back to Main](../../README.md) | [Next: Custom Endpoints →](../05-custom-endpoints/README.md)
