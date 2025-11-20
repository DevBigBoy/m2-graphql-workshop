# Real-World Examples

## Overview

This section provides complete, production-ready examples of GraphQL implementations in Magento 2.

## Topics Covered

1. [Complete Custom Module Example](01-complete-module/README.md)
2. [Blog Posts API](02-blog-api/README.md)
3. [Product Reviews Extension](03-product-reviews/README.md)
4. [Custom Wishlist Features](04-custom-wishlist/README.md)
5. [Store Locator with GraphQL](05-store-locator/README.md)
6. [Custom Customer Attributes](06-customer-attributes/README.md)
7. [Advanced Product Filtering](07-advanced-filtering/README.md)
8. [Subscription Management](08-subscriptions/README.md)
9. [Gift Registry System](09-gift-registry/README.md)
10. [Custom Checkout Fields](10-checkout-fields/README.md)

## Learning Objectives

By the end of this section, you will:

- Understand complete module structure
- See real-world resolver implementations
- Learn from production-ready code
- Understand common patterns and use cases
- Be able to build your own custom GraphQL features

## Example 1: Simple Blog API

Complete implementation of a blog posts GraphQL API.

### Module Structure

```text
app/code/Vendor/Blog/
├── etc/
│   ├── module.xml
│   ├── db_schema.xml
│   ├── schema.graphqls
│   └── di.xml
├── Model/
│   ├── Post.php
│   ├── ResourceModel/
│   │   ├── Post.php
│   │   └── Post/
│   │       └── Collection.php
│   ├── Resolver/
│   │   ├── Posts.php
│   │   ├── Post.php
│   │   ├── CreatePost.php
│   │   └── UpdatePost.php
│   └── DataProvider/
│       └── PostDataProvider.php
├── Api/
│   ├── Data/
│   │   └── PostInterface.php
│   └── PostRepositoryInterface.php
└── composer.json
```

### Schema Definition

```graphql
# etc/schema.graphqls
type Query {
    blogPosts(
        filter: BlogPostFilterInput
        pageSize: Int = 20
        currentPage: Int = 1
    ): BlogPostsOutput @resolver(class: "Vendor\\Blog\\Model\\Resolver\\Posts") @doc(description: "Get blog posts")

    blogPost(id: Int!): BlogPost @resolver(class: "Vendor\\Blog\\Model\\Resolver\\Post") @doc(description: "Get blog post by ID")
}

type Mutation {
    createBlogPost(input: CreateBlogPostInput!): BlogPost @resolver(class: "Vendor\\Blog\\Model\\Resolver\\CreatePost") @doc(description: "Create a new blog post")

    updateBlogPost(id: Int!, input: UpdateBlogPostInput!): BlogPost @resolver(class: "Vendor\\Blog\\Model\\Resolver\\UpdatePost") @doc(description: "Update blog post")
}

input BlogPostFilterInput {
    title: FilterTypeInput
    status: FilterTypeInput
    author_id: FilterTypeInput
}

input CreateBlogPostInput {
    title: String!
    content: String!
    status: String
    author_id: Int
}

input UpdateBlogPostInput {
    title: String
    content: String
    status: String
}

type BlogPostsOutput {
    items: [BlogPost]
    total_count: Int
    page_info: SearchResultPageInfo
}

type BlogPost {
    id: Int
    title: String
    content: String
    status: String
    author_id: Int
    author_name: String @resolver(class: "Vendor\\Blog\\Model\\Resolver\\Post\\AuthorName")
    created_at: String
    updated_at: String
}
```

### Resolver: Get Posts

```php
<?php
namespace Vendor\Blog\Model\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Query\ResolverInterface;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;
use Magento\Framework\GraphQl\Exception\GraphQlInputException;
use Vendor\Blog\Model\ResourceModel\Post\CollectionFactory;

class Posts implements ResolverInterface
{
    private $collectionFactory;

    public function __construct(CollectionFactory $collectionFactory)
    {
        $this->collectionFactory = $collectionFactory;
    }

    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) {
        $pageSize = $args['pageSize'] ?? 20;
        $currentPage = $args['currentPage'] ?? 1;

        $collection = $this->collectionFactory->create();

        // Apply filters
        if (isset($args['filter'])) {
            $this->applyFilters($collection, $args['filter']);
        }

        // Set pagination
        $collection->setPageSize($pageSize);
        $collection->setCurPage($currentPage);

        $items = [];
        foreach ($collection as $post) {
            $items[] = [
                'id' => $post->getId(),
                'title' => $post->getTitle(),
                'content' => $post->getContent(),
                'status' => $post->getStatus(),
                'author_id' => $post->getAuthorId(),
                'created_at' => $post->getCreatedAt(),
                'updated_at' => $post->getUpdatedAt(),
                'model' => $post
            ];
        }

        return [
            'items' => $items,
            'total_count' => $collection->getSize(),
            'page_info' => [
                'page_size' => $pageSize,
                'current_page' => $currentPage,
                'total_pages' => ceil($collection->getSize() / $pageSize)
            ]
        ];
    }

    private function applyFilters($collection, array $filters)
    {
        foreach ($filters as $field => $condition) {
            foreach ($condition as $conditionType => $value) {
                $collection->addFieldToFilter($field, [$conditionType => $value]);
            }
        }
    }
}
```

### Resolver: Create Post

```php
<?php
namespace Vendor\Blog\Model\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Query\ResolverInterface;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;
use Magento\Framework\GraphQl\Exception\GraphQlInputException;
use Magento\Framework\GraphQl\Exception\GraphQlAuthorizationException;
use Vendor\Blog\Api\PostRepositoryInterface;
use Vendor\Blog\Model\PostFactory;

class CreatePost implements ResolverInterface
{
    private $postRepository;
    private $postFactory;

    public function __construct(
        PostRepositoryInterface $postRepository,
        PostFactory $postFactory
    ) {
        $this->postRepository = $postRepository;
        $this->postFactory = $postFactory;
    }

    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) {
        // Check authentication
        if (!$context->getUserId()) {
            throw new GraphQlAuthorizationException(
                __('You must be logged in to create a post')
            );
        }

        // Validate input
        if (!isset($args['input']['title']) || empty($args['input']['title'])) {
            throw new GraphQlInputException(__('Title is required'));
        }

        if (!isset($args['input']['content']) || empty($args['input']['content'])) {
            throw new GraphQlInputException(__('Content is required'));
        }

        try {
            // Create post
            $post = $this->postFactory->create();
            $post->setTitle($args['input']['title']);
            $post->setContent($args['input']['content']);
            $post->setStatus($args['input']['status'] ?? 'draft');
            $post->setAuthorId($args['input']['author_id'] ?? $context->getUserId());

            // Save post
            $savedPost = $this->postRepository->save($post);

            return [
                'id' => $savedPost->getId(),
                'title' => $savedPost->getTitle(),
                'content' => $savedPost->getContent(),
                'status' => $savedPost->getStatus(),
                'author_id' => $savedPost->getAuthorId(),
                'created_at' => $savedPost->getCreatedAt(),
                'updated_at' => $savedPost->getUpdatedAt()
            ];

        } catch (\Exception $e) {
            throw new GraphQlInputException(
                __('Unable to create post: %1', $e->getMessage())
            );
        }
    }
}
```

### Example Queries

#### Get All Posts

```graphql
{
  blogPosts(pageSize: 10, currentPage: 1) {
    items {
      id
      title
      content
      status
      author_name
      created_at
    }
    total_count
    page_info {
      page_size
      current_page
      total_pages
    }
  }
}
```

#### Get Single Post

```graphql
{
  blogPost(id: 1) {
    id
    title
    content
    status
    author_id
    author_name
    created_at
    updated_at
  }
}
```

#### Create Post

```graphql
mutation {
  createBlogPost(
    input: {
      title: "My First Blog Post"
      content: "This is the content of my first blog post."
      status: "published"
    }
  ) {
    id
    title
    content
    status
    created_at
  }
}
```

#### Filter Posts

```graphql
{
  blogPosts(
    filter: {
      status: { eq: "published" }
      title: { like: "%GraphQL%" }
    }
    pageSize: 5
  ) {
    items {
      id
      title
      status
    }
    total_count
  }
}
```

## Example 2: Product Review System

See [Product Reviews Extension](03-product-reviews/README.md) for complete implementation.

## Example 3: Custom Customer Attributes

See [Custom Customer Attributes](06-customer-attributes/README.md) for complete implementation.

## Code Organization Best Practices

### 1. Separate Concerns

- **Models**: Data structures and business logic
- **Resolvers**: GraphQL field resolution
- **Data Providers**: Data fetching and transformation
- **Services**: Reusable business operations

### 2. Use Dependency Injection

```php
public function __construct(
    PostRepositoryInterface $postRepository,
    SearchCriteriaBuilder $searchCriteriaBuilder,
    LoggerInterface $logger
) {
    $this->postRepository = $postRepository;
    $this->searchCriteriaBuilder = $searchCriteriaBuilder;
    $this->logger = $logger;
}
```

### 3. Implement Proper Error Handling

```php
try {
    $result = $this->processData($args);
    return $result;
} catch (NoSuchEntityException $e) {
    throw new GraphQlNoSuchEntityException(__($e->getMessage()));
} catch (\Exception $e) {
    $this->logger->critical($e);
    throw new GraphQlInputException(__('An error occurred'));
}
```

### 4. Use Type Hints and Return Types

```php
public function resolve(
    Field $field,
    $context,
    ResolveInfo $info,
    array $value = null,
    array $args = null
): array {
    // Implementation
}
```

## Testing Examples

See each example directory for corresponding test files.

## Next Steps

Explore each example in detail:

1. [Complete Module Example](01-complete-module/README.md)
2. [Blog API](02-blog-api/README.md)
3. [Product Reviews](03-product-reviews/README.md)

---

[← Previous: Debugging](../12-debugging/README.md) | [Back to Main](../../README.md) | [Next: Tools & Resources →](../14-tools-resources/README.md)
