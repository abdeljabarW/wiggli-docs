Create Aggregate Related Classes
-------

- [Create {Aggregate}Collection Repository Interface](#create-aggregatecollection-repository-interface)
- [Add {AGGREGATE}_COLLECTION Constant To **MongoDb** Collections](#add-aggregate_collection-constant-to-mongodb-collections)
- [Register The {Aggregate}Collection Repository Factory](#register-the-aggregatecollection-repository-factory)
- [Create {Aggregate}Id class](#create-aggregateid-class)
- [Create EventStore{Aggregate}Collection](#create-eventstoreaggregatecollection)
- [Create The Aggregate](#create-the-aggregate)
- [Link The Aggregate to EventStore{Aggregate}Collection](#link-the-aggregate-to-eventstoreaggregatecollection)
- [Link The Aggregate To The {Aggregate}Collection](#link-the-aggregate-to-the-aggregatecollection)
- [Create {Aggregate}Projector](#create-aggregateprojector)
- [Create {Aggregate}ProjectorFactory](#create-aggregateprojectorfactory)
- [Register The {Aggregate}Projector Factory](#register-the-aggregateprojector-factory)
- [Create {Aggregate}ProcessManager](#create-aggregateprocessmanager)
- [Create {Aggregate}ProcessManagerFactory](#create-aggregateprocessmanagerfactory)
- [Register {Aggregate}ProcessManager factory](#register-aggregateprocessmanager-factory)

## Create {Aggregate}Collection Repository Interface

```php
<?php

namespace Company\Model\Repository;

use Company\Model\Aggregate\Review;
use Company\Model\ValueObject\ReviewId;

interface ReviewCollection
{
    const CONTAINER_ID = 'review_collection';

    public function add(Review $review);

    public function get(ReviewId $reviewId);
}
```

## Add {AGGREGATE}_COLLECTION Constant To **MongoDb** Collections

```php
<?php

class Collections
{
  // Rest of code
  const REVIEW_COLLECTION = 'read_review';
}
```

## Register The {Aggregate}Collection Repository Factory

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    'service_manager' => [
        // rest of code
        'factories' => [
            // Model
            \Company\Model\Repository\ReviewCollection::class
            => \Company\Container\Infrastructure\Repository\EventStoreReviewCollectionFactory::class,
          ]
          // rest of code
    ],
    // rest of code
];
```

## Create {Aggregate}Id class

```php
<?php

namespace Company\Model\ValueObject;

use Rhumsaa\Uuid\Uuid;

final class ReviewId
{
    /**
     * @var Uuid
     */
    private $uuid;

    public function __construct(Uuid $uuid)
    {
        $this->uuid = $uuid;
    }

    public static function generate(): self
    {
        return new self(Uuid::uuid4());
    }

    public static function fromString(string $reviewId)
    {
        return new self(Uuid::fromString($reviewId));
    }

    public function toString(): string
    {
        return $this->uuid->toString();
    }

    public function sameValueAs(ReviewId $reviewIdToCompare): bool
    {
        return $this->toString() === $reviewIdToCompare->toString();
    }
}
```

## Create EventStore{Aggregate}Collection

```php
<?php

namespace Company\Infrastructure\Repository;

use Company\Model\Aggregate\Review;
use Company\Model\Repository\ReviewCollection;
use Company\Model\ValueObject\ReviewId;
use Prooph\EventStore\Aggregate\AggregateRepository;

class EventStoreReviewCollection extends AggregateRepository implements ReviewCollection
{
    public function add(Review $review)
    {
        $this->addAggregateRoot($review);
    }

    public function get(ReviewId $reviewId)
    {
        return $this->getAggregateRoot($reviewId->toString());
    }
}
```

## Create The Aggregate

```php
<?php

namespace Company\Model\Aggregate;

use Company\Model\Event\ReviewWasCreated;
use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateRoot;
use User\Model\ValueObject\UserId;

final class Review extends AggregateRoot
{
    /** @var ReviewId */
    protected $id;
    /** @var UserId */
    protected $userId;
    /** @var string */
    protected $freelancerId;
    /** @var string */
    protected $comment;

    /**
     * @return string
     */
    protected function aggregateId(): string
    {
        return $this->id->toString();
    }

    /**
     * @return UserId
     */
    public function userId(): UserId
    {
        return $this->userId;
    }

    /**
     * @return string
     */
    public function freelancerId(): string
    {
        return $this->freelancerId;
    }

    /**
     * @return string
     */
    public function comment(): string
    {
        return $this->comment;
    }
}
```

## Link The Aggregate to EventStore{Aggregate}Collection

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    // rest of code
    'prooph' => [
        'event_store' => [
            // rest of code
            \Company\Model\Repository\ReviewCollection::CONTAINER_ID => [
                'repository_class' => \Company\Infrastructure\Repository\EventStoreReviewCollection::class,
                'aggregate_type' => \Company\Model\Aggregate\Review::class,
                'aggregate_translator' => \Prooph\EventSourcing\EventStoreIntegration\AggregateTranslator::class,
                'snapshot_store' => \Prooph\EventStore\Snapshot\SnapshotStore::class,
            ],
            // rest of code
          ]
    ],
    // rest of code
];
```

## Link The Aggregate To The {Aggregate}Collection

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    // rest of code
    'prooph' => [
        'snapshotter' => [
            // rest of code
            'aggregate_repositories' => [
                // rest of code
                \Company\Model\Aggregate\Review::class => \Company\Model\Repository\ReviewCollection::class,
            ]
            // rest of code
          ]
    ],
    // rest of code
];
```

## Create {Aggregate}Projector

```php
<?php

namespace Company\Projection;

use Common\Application\Base\Traits\MongoDbTrait;
use Common\Container\Infrastructure\MongoDb\Collections;
use MongoDB\Client;

final class ReviewProjector
{
    use MongoDbTrait;

    /**
     * @var Client
     */
    private $mongoClient;

    /**
     * @var \MongoCollection
     */
    private $mainCollection;

    /**
     * @param Client $mongoClient
     */
    public function __construct(Client $mongoClient)
    {
        $this->mongoClient = $mongoClient;
        $this->mainCollection = $this->getCollection(Collections::REVIEW_COLLECTION);
    }
}
```

## Create {Aggregate}ProjectorFactory

```php
<?php

namespace Company\Container\Projection;

use Company\Projection\ReviewProjector;
use MongoDB\Client;
use Zend\ServiceManager\ServiceManager;

class ReviewProjectorFactory
{
    /**
     * @param ServiceManager $container
     * @return ReviewProjector
     */
    public function __invoke(ServiceManager $container): ReviewProjector
    {
        return new ReviewProjector($container->get(Client::class));
    }
}
```

## Register The {Aggregate}Projector Factory

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    'service_manager' => [
        // rest of code
        'factories' => [
            // Projections
            \Company\Projection\ReviewProjector::class
            => \Company\Container\Projection\ReviewProjectorFactory::class,
          ]
          // rest of code
    ],
    // rest of code
];
```

## Create {Aggregate}ProcessManager

```php
<?php

namespace Company\ProcessManager;

use Company\Projection\ReviewProjector;
use Company\Model\Event\ReviewWasCreated;

class ReviewProcessManager
{
    const ID = '_id';
    const USER_ID = 'user_id';
    const FREELANCER_ID = 'freelancer_id';
    const COMMENT = 'comment';

    /**
     * @var ReviewProjector
     */
    private $reviewProjector;

    public function __construct(ReviewProjector $reviewProjector)
    {
        $this->reviewProjector = $reviewProjector;
    }
}
```

## Create {Aggregate}ProcessManagerFactory

```php
<?php

namespace Company\Container\ProcessManager;

use Company\ProcessManager\ReviewProcessManager;
use Company\Projection\ReviewProjector;
use Zend\ServiceManager\ServiceManager;

final class ReviewProcessManagerFactory
{
    /**
     * @param ServiceManager $container
     * @return ReviewProcessManager
     */
    public function __invoke(ServiceManager $container)
    {
        return new ReviewProcessManager($container->get(ReviewProjector::class));
    }
}
```

## Register {Aggregate}ProcessManager factory

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    'service_manager' => [
        // rest of code
        'factories' => [
            // Projections
            \Company\ProcessManager\ReviewProcessManager::class
            => \Company\Container\ProcessManager\ReviewProcessManagerFactory::class,
        ]
        // rest of code
    ],
    // rest of code
];
```