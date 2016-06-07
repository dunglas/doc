# Data providers

To retrieve data exposed by the API, API Platform uses classes called **data providers**. A data provider using [Doctrine
ORM](http://www.doctrine-project.org/projects/orm.html) to retrieve data from a database is included with the library and
is enabled by default. This data provider natively supports paged collections and filters. It can be used as is and fits
perfectly with common usages.

But sometime, you want to retrieve data from other sources such as a webservice, ElasticSearch, MongoDB or another ORM.
Custom data providers can be used to do so. A project can include as many data providers as it needs. The first able to retrieve data for a given resource will be used.

For a given resource, you can implement two kind of data providers:
- the [CollectionDataProvider](https://github.com/api-platform/core/blob/master/src/Api/CollectionDataProviderInterface.php) is used when fetching collection operations
- the [ItemDataProvider](https://github.com/api-platform/core/blob/master/src/Api/ItemDataProviderInterface.php) is used when fetching item operations

In the following examples we will custom data providers for the class `Acme\Model\BlogPost`.

### Custom collection data provider

First declare a symfony service, for example:

```
# Acme\Resources\config\services.yml
services:
    blog_post.collection_data_provider:
        class: 'Acme\DataProvider\BlogCollectionDataProvider'
        # data providers are using decorators (http://symfony.com/doc/master//components/dependency_injection/advanced.html#decorating-services)
        decorates: 'api_platform.collection_data_provider'
        arguments:
            - '@blog_post.collection_data_provider.inner'
```

Then, your `BlogCollectionDataProvider` has to implement the [`CollectionDataProviderInterface`](https://github.com/api-platform/core/blob/master/src/Api/CollectionDataProviderInterface.php):

The `getCollection` method must return an `array`, or a `\Traversable` instance. If no data is available, you should return an empty array.

```
<?php

namespace Acme\DataProvider;

use ApiPlatform\Core\Api\CollectionDataProviderInterface;
use ApiPlatform\Core\Exception\ResourceClassNotSupportedException;

class BlogCollectionDataProvider implements CollectionDataProviderInterface
{
    private $decorated;

    /**
     * @param CollectionDataProviderInterface|null $decorated
     */
    public function __construct(CollectionDataProviderInterface $decorated = null)
    {
        $this->decorated = $decorated;
    }

    /**
     * {@inheritdoc}
     */
    public function getCollection(string $resourceClass, string $operationName = null)
    {
        // tests if the class is supported by this data provider
        if ('Acme\Model\BlogPost' !== $resourceClass) {
          throw new \ResourceClassNotSupportedException();
        }

        return ['foo' => 'bar'];
    }
}
```

### Custom item data provider

Declare a symfony service, for example:

```
# Acme\Resources\config\services.yml
services:
    blog_post.item_data_provider:
        class: 'Acme\DataProvider\BlogItemDataProvider'
        decorates: 'api_platform.item_data_provider'
        arguments:
            - '@blog_post.item_data_provider.inner'
```

Then, your `BlogItemDataProvider` has to implement the [`ItemDataProviderInterface`](https://github.com/api-platform/core/blob/master/src/Api/ItemDataProviderInterface.php):

The `getItem` method can return `null` if no result has been found.

```
<?php

namespace Acme\DataProvider;

use ApiPlatform\Core\Api\ItemDataProviderInterface;
use ApiPlatform\Core\Exception\ResourceClassNotSupportedException;

class BlogItemDataProvider implements ItemDataProviderInterface
{
    private $decorated;

    /**
     * @param CollectionDataProviderInterface|null $decorated
     */
    public function __construct(ItemDataProviderInterface $decorated = null)
    {
        $this->decorated = $decorated;
    }

    /**
     * {@inheritdoc}
     */
    public function getCollection(string $resourceClass, string $operationName = null)
    {
        if ('Acme\Model\BlogPost' !== $resourceClass) {
          throw new \ResourceClassNotSupportedException();
        }

        return (object) ['foo' => 'bar'];
    }
}
```

Previous chapter: [Operations](operations.md)<br>
Next chapter: [Filters](filters.md)
