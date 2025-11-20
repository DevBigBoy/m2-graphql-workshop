# GraphQL Fragments

## Overview

Fragments are reusable units in GraphQL that allow you to construct sets of fields and reuse them across multiple queries and mutations.

## What are Fragments?

Fragments let you define a set of fields once and reuse them in multiple places. This is particularly useful when you need to fetch the same fields from the same type in different queries.

## Basic Fragment Syntax

```graphql
fragment FragmentName on TypeName {
  field1
  field2
  field3
}
```

## Using Fragments in Magento GraphQL

### Example 1: Product Fragment

```graphql
fragment ProductDetails on ProductInterface {
  id
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
  image {
    url
    label
  }
}

# Using the fragment in a query
query GetProducts {
  products(filter: { category_id: { eq: "3" } }) {
    items {
      ...ProductDetails
    }
  }
}
```

### Example 2: Customer Address Fragment

```graphql
fragment AddressDetails on CustomerAddress {
  id
  firstname
  lastname
  street
  city
  region {
    region_code
    region
  }
  postcode
  country_code
  telephone
}

query GetCustomerAddresses {
  customer {
    addresses {
      ...AddressDetails
    }
    default_shipping {
      ...AddressDetails
    }
    default_billing {
      ...AddressDetails
    }
  }
}
```

### Example 3: Price Fragment

```graphql
fragment PriceInfo on ProductPrice {
  regularPrice {
    amount {
      value
      currency
    }
  }
  finalPrice {
    amount {
      value
      currency
    }
  }
  discount {
    amount_off
    percent_off
  }
}

query GetProductWithPrice($sku: String!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      name
      sku
      price_range {
        minimum_price {
          ...PriceInfo
        }
        maximum_price {
          ...PriceInfo
        }
      }
    }
  }
}
```

## Nested Fragments

Fragments can include other fragments:

```graphql
fragment MediaGallery on ProductInterface {
  media_gallery {
    url
    label
    position
  }
}

fragment FullProductDetails on ProductInterface {
  id
  sku
  name
  description {
    html
  }
  ...MediaGallery
  price_range {
    minimum_price {
      ...PriceInfo
    }
  }
}
```

## Inline Fragments

Use inline fragments when working with interfaces or unions:

```graphql
query GetProducts {
  products(filter: { sku: { eq: "24-MB01" } }) {
    items {
      id
      sku
      name
      ... on ConfigurableProduct {
        configurable_options {
          id
          label
          values {
            value_index
            label
          }
        }
      }
      ... on BundleProduct {
        items {
          title
          options {
            label
            quantity
          }
        }
      }
    }
  }
}
```

## Benefits of Using Fragments

### 1. Code Reusability

```graphql
# Without fragments (repetitive)
query GetMultipleProducts {
  product1: products(filter: { sku: { eq: "SKU1" } }) {
    items {
      id
      sku
      name
      price { regularPrice { amount { value } } }
    }
  }
  product2: products(filter: { sku: { eq: "SKU2" } }) {
    items {
      id
      sku
      name
      price { regularPrice { amount { value } } }
    }
  }
}

# With fragments (DRY principle)
fragment BasicProduct on ProductInterface {
  id
  sku
  name
  price { regularPrice { amount { value } } }
}

query GetMultipleProducts {
  product1: products(filter: { sku: { eq: "SKU1" } }) {
    items {
      ...BasicProduct
    }
  }
  product2: products(filter: { sku: { eq: "SKU2" } }) {
    items {
      ...BasicProduct
    }
  }
}
```

### 2. Consistency

Fragments ensure that the same fields are requested across different parts of your application.

### 3. Maintainability

Update fields in one place (the fragment) instead of multiple queries.

## Fragment Variables

Fragments can access query variables:

```graphql
fragment ProductList on Products {
  items {
    id
    sku
    name
  }
  page_info {
    page_size
    current_page
    total_pages
  }
}

query GetProducts($pageSize: Int = 20, $currentPage: Int = 1) {
  products(
    filter: { category_id: { eq: "3" } }
    pageSize: $pageSize
    currentPage: $currentPage
  ) {
    ...ProductList
  }
}
```

## Complex Example: Cart with Fragments

```graphql
fragment MoneyFields on Money {
  value
  currency
}

fragment CartPriceFields on CartPrices {
  grand_total {
    ...MoneyFields
  }
  subtotal_excluding_tax {
    ...MoneyFields
  }
  subtotal_including_tax {
    ...MoneyFields
  }
}

fragment CartItemFields on CartItemInterface {
  id
  quantity
  product {
    name
    sku
  }
  prices {
    price {
      ...MoneyFields
    }
    row_total {
      ...MoneyFields
    }
  }
}

fragment CartFields on Cart {
  id
  email
  total_quantity
  items {
    ...CartItemFields
  }
  prices {
    ...CartPriceFields
  }
}

query GetCart($cartId: String!) {
  cart(cart_id: $cartId) {
    ...CartFields
  }
}

mutation AddToCart($cartId: String!, $sku: String!, $quantity: Float!) {
  addSimpleProductsToCart(
    input: {
      cart_id: $cartId
      cart_items: [{ data: { sku: $sku, quantity: $quantity } }]
    }
  ) {
    cart {
      ...CartFields
    }
  }
}
```

## Best Practices

### 1. Name Fragments Descriptively

```graphql
# Good
fragment ProductBasicInfo on ProductInterface { ... }
fragment ProductPricingInfo on ProductInterface { ... }
fragment ProductMediaGallery on ProductInterface { ... }

# Bad
fragment Fragment1 on ProductInterface { ... }
fragment Data on ProductInterface { ... }
```

### 2. Keep Fragments Focused

```graphql
# Good - focused fragments
fragment ProductIdentity on ProductInterface {
  id
  sku
  name
}

fragment ProductPricing on ProductInterface {
  price_range {
    minimum_price {
      regular_price { value currency }
    }
  }
}

# Bad - too many unrelated fields
fragment Everything on ProductInterface {
  id
  sku
  name
  price_range { ... }
  description { ... }
  media_gallery { ... }
  related_products { ... }
  # Too much in one fragment
}
```

### 3. Use Fragment Composition

```graphql
fragment ProductCore on ProductInterface {
  id
  sku
  name
}

fragment ProductDisplay on ProductInterface {
  ...ProductCore
  image {
    url
    label
  }
  small_image {
    url
    label
  }
}

fragment ProductFull on ProductInterface {
  ...ProductDisplay
  description {
    html
  }
  short_description {
    html
  }
  media_gallery {
    url
    label
  }
}
```

### 4. Co-locate Fragments with Components (in React/Vue)

```javascript
// ProductCard.jsx
import { gql } from '@apollo/client';

const PRODUCT_CARD_FRAGMENT = gql`
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
    image {
      url
      label
    }
  }
`;

export { PRODUCT_CARD_FRAGMENT };
```

## Fragment Type Conditions

Fragments must be defined on a specific type:

```graphql
# Valid - fragment on interface
fragment ProductFields on ProductInterface {
  id
  sku
  name
}

# Valid - fragment on concrete type
fragment SimpleProductFields on SimpleProduct {
  id
  sku
  name
}

# Usage with type checking
query GetProduct($sku: String!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      ...ProductFields
      ... on SimpleProduct {
        ...SimpleProductFields
      }
      ... on ConfigurableProduct {
        configurable_options {
          id
          label
        }
      }
    }
  }
}
```

## Common Use Cases in Magento

### 1. Product Listing Pages

```graphql
fragment PLPProduct on ProductInterface {
  id
  sku
  name
  url_key
  small_image {
    url
  }
  price_range {
    minimum_price {
      regular_price {
        value
        currency
      }
      final_price {
        value
      }
    }
  }
}
```

### 2. Product Detail Pages

```graphql
fragment PDPProduct on ProductInterface {
  id
  sku
  name
  description {
    html
  }
  media_gallery {
    url
    label
  }
  price_range {
    minimum_price {
      regular_price {
        value
        currency
      }
    }
  }
  ... on ConfigurableProduct {
    configurable_options {
      id
      label
      values {
        value_index
        label
        swatch_data {
          value
        }
      }
    }
  }
}
```

### 3. Checkout Flow

```graphql
fragment CheckoutAddress on CartAddressInterface {
  firstname
  lastname
  street
  city
  region {
    code
    label
  }
  postcode
  country {
    code
    label
  }
  telephone
}

fragment CheckoutCart on Cart {
  id
  email
  billing_address {
    ...CheckoutAddress
  }
  shipping_addresses {
    ...CheckoutAddress
    selected_shipping_method {
      carrier_code
      method_code
      amount {
        value
        currency
      }
    }
  }
}
```

## Troubleshooting

### Fragment Not Found

```graphql
# Error: Fragment "ProductDetails" not found
query GetProducts {
  products {
    items {
      ...ProductDetails  # Fragment not defined
    }
  }
}

# Solution: Define the fragment
fragment ProductDetails on ProductInterface {
  id
  sku
  name
}

query GetProducts {
  products {
    items {
      ...ProductDetails
    }
  }
}
```

### Wrong Fragment Type

```graphql
# Error: Fragment cannot be spread here
fragment CustomerFields on Customer {
  firstname
  lastname
}

query GetProducts {
  products {
    items {
      ...CustomerFields  # Wrong type!
    }
  }
}

# Solution: Use correct type
fragment ProductFields on ProductInterface {
  name
  sku
}

query GetProducts {
  products {
    items {
      ...ProductFields
    }
  }
}
```

## Performance Considerations

1. **Don't over-fetch**: Include only necessary fields in fragments
2. **Avoid deep nesting**: Keep fragment depth reasonable
3. **Reuse wisely**: Balance reusability with specificity
4. **Monitor query size**: Large fragments can create heavy queries

---

[← Previous: Using Variables](08-using-variables.md) | [Back to Getting Started](README.md) | [Next: Architecture →](../03-architecture/README.md)
