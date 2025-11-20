# Testing GraphQL

## Overview

Learn how to test your GraphQL implementation effectively using various testing strategies and tools.

## Topics Covered

1. [Testing Strategy Overview](01-overview.md)
2. [Integration Tests](02-integration-tests.md)
   - Magento test framework
   - Creating GraphQL integration tests
   - Test fixtures and data
3. [Unit Testing Resolvers](03-unit-tests.md)
   - Mocking dependencies
   - Testing resolver logic
4. [API Testing](04-api-testing.md)
   - Postman collections
   - Automated testing with Newman
   - GraphQL-specific testing tools
5. [Testing Authentication](05-testing-auth.md)
6. [Testing Mutations](06-testing-mutations.md)
7. [Performance Testing](07-performance-testing.md)
   - Load testing
   - Query performance benchmarks
8. [Continuous Integration](08-ci-cd.md)
9. [Test Data Management](09-test-data.md)

## Learning Objectives

By the end of this section, you will:

- Write integration tests for GraphQL endpoints
- Create unit tests for resolvers
- Use API testing tools
- Test authentication flows
- Perform performance testing
- Set up CI/CD for GraphQL tests

## Integration Test Example

```php
<?php
namespace Vendor\Module\Test\Integration\GraphQl;

use Magento\TestFramework\TestCase\GraphQlAbstract;

class ProductQueryTest extends GraphQlAbstract
{
    /**
     * @magentoApiDataFixture Magento/Catalog/_files/product_simple.php
     */
    public function testGetProductBySku()
    {
        $sku = 'simple';

        $query = <<<QUERY
{
    products(filter: {sku: {eq: "$sku"}}) {
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
QUERY;

        $response = $this->graphQlQuery($query);

        $this->assertArrayHasKey('products', $response);
        $this->assertArrayHasKey('items', $response['products']);
        $this->assertNotEmpty($response['products']['items']);

        $product = $response['products']['items'][0];
        $this->assertEquals($sku, $product['sku']);
        $this->assertArrayHasKey('name', $product);
        $this->assertArrayHasKey('price', $product);
    }

    /**
     * @magentoApiDataFixture Magento/Customer/_files/customer.php
     */
    public function testGetCustomerDataRequiresAuth()
    {
        $query = <<<QUERY
{
    customer {
        firstname
        lastname
        email
    }
}
QUERY;

        $this->expectException(\Exception::class);
        $this->expectExceptionMessage('The current customer isn\'t authorized');

        $this->graphQlQuery($query);
    }

    /**
     * @magentoApiDataFixture Magento/Customer/_files/customer.php
     */
    public function testGetCustomerDataWithAuth()
    {
        $email = 'customer@example.com';
        $password = 'password';

        $query = <<<QUERY
mutation {
    generateCustomerToken(email: "$email", password: "$password") {
        token
    }
}
QUERY;

        $response = $this->graphQlMutation($query);
        $token = $response['generateCustomerToken']['token'];

        $customerQuery = <<<QUERY
{
    customer {
        firstname
        lastname
        email
    }
}
QUERY;

        $headers = ['Authorization' => 'Bearer ' . $token];
        $response = $this->graphQlQuery($customerQuery, [], '', $headers);

        $this->assertArrayHasKey('customer', $response);
        $this->assertEquals($email, $response['customer']['email']);
    }
}
```

## Unit Test Example

```php
<?php
namespace Vendor\Module\Test\Unit\Model\Resolver;

use PHPUnit\Framework\TestCase;
use Vendor\Module\Model\Resolver\CustomResolver;

class CustomResolverTest extends TestCase
{
    private $resolver;
    private $dataProvider;
    private $field;
    private $context;
    private $info;

    protected function setUp(): void
    {
        $this->dataProvider = $this->createMock(\Vendor\Module\Model\DataProvider::class);
        $this->field = $this->createMock(\Magento\Framework\GraphQl\Config\Element\Field::class);
        $this->context = $this->createMock(\Magento\Framework\GraphQl\Query\Resolver\ContextInterface::class);
        $this->info = $this->createMock(\Magento\Framework\GraphQl\Schema\Type\ResolveInfo::class);

        $this->resolver = new CustomResolver($this->dataProvider);
    }

    public function testResolveReturnsData()
    {
        $expectedData = [
            'id' => 1,
            'name' => 'Test Item'
        ];

        $this->dataProvider
            ->expects($this->once())
            ->method('getData')
            ->with(1)
            ->willReturn($expectedData);

        $args = ['id' => 1];
        $result = $this->resolver->resolve($this->field, $this->context, $this->info, null, $args);

        $this->assertEquals($expectedData, $result);
    }

    public function testResolveThrowsExceptionOnInvalidInput()
    {
        $this->expectException(\Magento\Framework\GraphQl\Exception\GraphQlInputException::class);

        $args = []; // Missing required 'id'
        $this->resolver->resolve($this->field, $this->context, $this->info, null, $args);
    }
}
```

## Postman Collection Example

```json
{
  "info": {
    "name": "Magento GraphQL Tests",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Get Store Config",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "graphql",
          "graphql": {
            "query": "{\n  storeConfig {\n    store_name\n    store_code\n    locale\n  }\n}",
            "variables": ""
          }
        },
        "url": {
          "raw": "{{base_url}}/graphql",
          "host": ["{{base_url}}"],
          "path": ["graphql"]
        }
      }
    },
    {
      "name": "Generate Customer Token",
      "event": [
        {
          "listen": "test",
          "script": {
            "exec": [
              "var jsonData = pm.response.json();",
              "pm.environment.set('customer_token', jsonData.data.generateCustomerToken.token);"
            ]
          }
        }
      ],
      "request": {
        "method": "POST",
        "header": [],
        "body": {
          "mode": "graphql",
          "graphql": {
            "query": "mutation {\n  generateCustomerToken(email: \"{{customer_email}}\", password: \"{{customer_password}}\") {\n    token\n  }\n}",
            "variables": ""
          }
        },
        "url": {
          "raw": "{{base_url}}/graphql",
          "host": ["{{base_url}}"],
          "path": ["graphql"]
        }
      }
    }
  ]
}
```

## Testing Best Practices

### 1. Use Data Fixtures

```php
/**
 * @magentoApiDataFixture Magento/Catalog/_files/product_simple.php
 * @magentoApiDataFixture Magento/Customer/_files/customer.php
 */
public function testSomething()
{
    // Test code
}
```

### 2. Test Both Success and Failure Cases

```php
public function testValidInput()
{
    // Test successful case
}

public function testInvalidInput()
{
    $this->expectException(GraphQlInputException::class);
    // Test failure case
}
```

### 3. Test Authentication and Authorization

```php
public function testRequiresAuthentication()
{
    // Test without token - should fail
}

public function testWithAuthentication()
{
    // Test with valid token - should succeed
}
```

### 4. Test Pagination

```php
public function testPagination()
{
    // Test first page
    // Test last page
    // Test page size limits
}
```

### 5. Verify Response Structure

```php
$this->assertArrayHasKey('data', $response);
$this->assertArrayNotHasKey('errors', $response);
```

## Running Tests

### Integration Tests

```bash
# Run all GraphQL integration tests
vendor/bin/phpunit -c dev/tests/integration/phpunit.xml.dist app/code/Vendor/Module/Test/Integration/

# Run specific test class
vendor/bin/phpunit -c dev/tests/integration/phpunit.xml.dist app/code/Vendor/Module/Test/Integration/GraphQl/ProductQueryTest.php
```

### Unit Tests

```bash
# Run all unit tests
vendor/bin/phpunit -c dev/tests/unit/phpunit.xml.dist app/code/Vendor/Module/Test/Unit/

# Run specific test class
vendor/bin/phpunit -c dev/tests/unit/phpunit.xml.dist app/code/Vendor/Module/Test/Unit/Model/Resolver/CustomResolverTest.php
```

### API Tests with Newman

```bash
# Install Newman
npm install -g newman

# Run Postman collection
newman run magento-graphql-tests.json -e environment.json

# Run with reporters
newman run magento-graphql-tests.json -e environment.json -r cli,json,html
```

## Next Steps

Start with [Testing Strategy Overview](01-overview.md) to plan your testing approach.

---

[← Previous: Advanced Topics](../09-advanced-topics/README.md) | [Back to Main](../../README.md) | [Next: Security →](../11-security/README.md)
