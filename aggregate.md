[The Aggregate](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CQRSI)
1. [Create repository ReviewCollection interface](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreaterepositoryReviewCollectioninterface)
2. [Add REVIEW\_COLLECTION constant to MongoDb Collections](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddREVIEW_COLLECTIONconstanttoMongoDbCollections)
3. [Register the ReviewCollection repository factory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-RegistertheReviewCollectionrepositoryfactory)
4. [Create ReviewId class](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateReviewIdclass)
5. [Create EventStoreReviewCollection](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateEventStoreReviewCollection)
6. [Create the aggregate Review](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreatetheaggregateReview)
7. [Link Review aggregate to EventStoreReviewCollection in the prooph.event\_store array](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-LinkReviewaggregatetoEventStoreReviewCollectionintheprooph.event_storearray)
8. [Link Review aggregate to ReviewCollection in prooph.snapshotter.aggregate\_repositories](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-LinkReviewaggregatetoReviewCollectioninprooph.snapshotter.aggregate_repositories)

### Create repository **ReviewCollection** interface

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

### Add **REVIEW\_COLLECTION** constant to **MongoDb** Collections

```php
<?php

class Collections
{
  // Rest of code
  const REVIEW_COLLECTION = 'read_review';
}
```

### Register the **ReviewCollection repository factory**

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

### Create **ReviewId** class

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

### Create **EventStoreReviewCollection**

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

### Create the aggregate **Review**

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

### Link Review aggregate to EventStoreReviewCollection in the prooph.event\_store array

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

### Link Review aggregate to ReviewCollection in prooph.snapshotter.aggregate\_repositories

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
