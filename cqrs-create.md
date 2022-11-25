*   1 [The Aggregate](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CQRSI)
    *   1.1 [Create repository ReviewCollection interface](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreaterepositoryReviewCollectioninterface)
    *   1.2 [Add REVIEW\_COLLECTION constant to MongoDb Collections](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddREVIEW_COLLECTIONconstanttoMongoDbCollections)
    *   1.3 [Register the ReviewCollection repository factory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-RegistertheReviewCollectionrepositoryfactory)
    *   1.4 [Create ReviewId class](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateReviewIdclass)
    *   1.5 [Create EventStoreReviewCollection](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateEventStoreReviewCollection)
    *   1.6 [Create the aggregate Review](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreatetheaggregateReview)
    *   1.7 [Link Review aggregate to EventStoreReviewCollection in the prooph.event\_store array](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-LinkReviewaggregatetoEventStoreReviewCollectionintheprooph.event_storearray)
    *   1.8 [Link Review aggregate to ReviewCollection in prooph.snapshotter.aggregate\_repositories](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-LinkReviewaggregatetoReviewCollectioninprooph.snapshotter.aggregate_repositories)
*   2 [CQRS-ES](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CQRSII)
    *   2.1 [Create CreateReview command](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateCreateReviewcommand)
    *   2.2 [Create CreateReviewHandler](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateCreateReviewHandler)
    *   2.3 [Create CreateReviewHandlerFactory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateCreateReviewHandlerFactory)
    *   2.4 [Register the CreateReviewHandler factory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-RegistertheCreateReviewHandlerfactory)
    *   2.5 [Link CreateReview command to the CreateReviewHandler](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-LinkCreateReviewcommandtotheCreateReviewHandler)
    *   2.6 [Create ReviewWasCreated event](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateReviewWasCreatedevent)
    *   2.7 [Register the ReviewWasCreated event](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-RegistertheReviewWasCreatedevent)
    *   2.8 [Add createReviewWithData method to Review aggregate](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddcreateReviewWithDatamethodtoReviewaggregate)
    *   2.9 [Add whenReviewWasCreated method to Review aggregate](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddwhenReviewWasCreatedmethodtoReviewaggregate)
    *   2.10 [Create ReviewProjector](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateReviewProjector)
    *   2.11 [Create ReviewProjectorFactory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateReviewProjectorFactory)
    *   2.12 [Register the ReviewProjector factory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-RegistertheReviewProjectorfactory)
    *   2.13 [Create ReviewProcessManager](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateReviewProcessManager)
    *   2.14 [Create ReviewProcessManagerFactory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-CreateReviewProcessManagerFactory)
    *   2.15 [Register ReviewProcessManager factory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-RegisterReviewProcessManagerfactory)
    *   2.16 [Add onReviewWasCreated to ReviewProcessManager](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddonReviewWasCreatedtoReviewProcessManager)
    *   2.17 [Link Event ReviewWasCreated to ReviewProcessManager](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-LinkEventReviewWasCreatedtoReviewProcessManager)
    *   2.18 [Add createReviewWithData and whenReviewWasCreated method to Review aggregate](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddcreateReviewWithDataandwhenReviewWasCreatedmethodtoReviewaggregate)
    *   2.19 [Add OnReviewWasCreated method to ReviewProcessManager](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddOnReviewWasCreatedmethodtoReviewProcessManager)
    *   2.20 [Create create\_review.php filter](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-Createcreate_review.phpfilter)
    *   2.21 [Add CREATE\_REVIEW\_FILTER to the FilterService class](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-AddCREATE_REVIEW_FILTERtotheFilterServiceclass)
    *   2.22 [Register create\_review filter](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-Registercreate_reviewfilter)
*   3 [Edit The API](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-EditTheAPI)
    *   3.1 [Edit CreateReviewResource](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-EditCreateReviewResource)
    *   3.2 [Edit CreateReviewResourceFactory](#CQRS+Apigilty-HowToAddACreateEndpoint,CommandsAndEvents-EditCreateReviewResourceFactory)

The API
-------

On the Apigility App, create a new service: **CreateReview**

The following files are generated during the process:

### CreateReviewEntity (as generated)

```php
<?php
namespace CompanyAPI\V1\Rest\CreateReview;

class CreateReviewEntity
{
}
```

### CreateReviewCollection (as generated)

```php
<?php
namespace CompanyAPI\V1\Rest\CreateReview;

use Zend\Paginator\Paginator;

class CreateReviewCollection extends Paginator
{
}
```

### CreateReviewResource (as generated)

```php
<?php
namespace CompanyAPI\V1\Rest\CreateReview;

use ZF\ApiProblem\ApiProblem;
use ZF\Rest\AbstractResourceListener;

class CreateReviewResource extends AbstractResourceListener
{
    /**
     * Create a resource
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function create($data)
    {
        return new ApiProblem(405, 'The POST method has not been defined');
    }

    /**
     * Delete a resource
     *
     * @param  mixed $id
     * @return ApiProblem|mixed
     */
    public function delete($id)
    {
        return new ApiProblem(405, 'The DELETE method has not been defined for individual resources');
    }

    /**
     * Delete a collection, or members of a collection
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function deleteList($data)
    {
        return new ApiProblem(405, 'The DELETE method has not been defined for collections');
    }

    /**
     * Fetch a resource
     *
     * @param  mixed $id
     * @return ApiProblem|mixed
     */
    public function fetch($id)
    {
        return new ApiProblem(405, 'The GET method has not been defined for individual resources');
    }

    /**
     * Fetch all or a subset of resources
     *
     * @param  array $params
     * @return ApiProblem|mixed
     */
    public function fetchAll($params = [])
    {
        return new ApiProblem(405, 'The GET method has not been defined for collections');
    }

    /**
     * Patch (partial in-place update) a resource
     *
     * @param  mixed $id
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function patch($id, $data)
    {
        return new ApiProblem(405, 'The PATCH method has not been defined for individual resources');
    }

    /**
     * Patch (partial in-place update) a collection or members of a collection
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function patchList($data)
    {
        return new ApiProblem(405, 'The PATCH method has not been defined for collections');
    }

    /**
     * Replace a collection or members of a collection
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function replaceList($data)
    {
        return new ApiProblem(405, 'The PUT method has not been defined for collections');
    }

    /**
     * Update a resource
     *
     * @param  mixed $id
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function update($id, $data)
    {
        return new ApiProblem(405, 'The PUT method has not been defined for individual resources');
    }
}
```

### CreateReviewResourceFactory (as generated)

```php
<?php
namespace CompanyAPI\V1\Rest\CreateReview;

class CreateReviewResourceFactory
{
    public function __invoke($services)
    {
        return new CreateReviewResource();
    }
}
```

CQRS I
------

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

CQRS II
-------

### Create **CreateReview** command

```php
<?php

namespace Company\Model\Command;

use Prooph\Common\Messaging\Command;
use Prooph\Common\Messaging\PayloadConstructable;
use Prooph\Common\Messaging\PayloadTrait;
use User\Model\ValueObject\UserId;

class CreateReview extends Command implements PayloadConstructable
{
    use PayloadTrait;

    /**
     * @param UserId $userId
     * @param string $freelancerId
     * @param $comment
     * @return CreateReview
     */
    public function withData(UserId $userId, string $freelancerId, $comment): CreateReview
    {
        return new self(
            [
                'user_id' => $userId,
                'freelancer_id' => $freelancerId,
                'comment' => $comment,
            ]
        );
    }

    /**
     * @return UserId
     */
    public function userId(): UserId
    {
        return UserId::fromString($this->payload['user_id']);
    }

    /**
     * @return mixed
     */
    public function freelancerId() {
        return $this->payload['freelancer_id'];
    }

    public function comment() {
        return $this->payload['comment'];
    }
}
```

### Create **CreateReviewHandler**

```php
<?php

namespace Company\Model\Command;

use Common\Application\Base\Traits\HireMeTrait;
use Company\Model\Aggregate\Review;
use Company\Model\Repository\ReviewCollection;
use Company\Model\ValueObject\ReviewId;
use User\Projection\UserFinder;

class CreateReviewHandler
{
    use HireMeTrait;

    /**
     * @var ReviewCollection
     */
    public $reviewCollection;

    public function __construct(ReviewCollection $reviewCollection)
    {
        $this->reviewCollection = $reviewCollection;
    }

    public function __invoke(CreateReview $createReviewCommand)
    {
        $review = Review::createReviewWithData(ReviewId::generate(), $createReviewCommand->userId(), $createReviewCommand->freelancerId(), $createReviewCommand->comment());
        $this->reviewCollection->add($review);
    }
}
```

### Create **CreateReviewHandlerFactory**

```php
<?php

namespace Company\Container\Model;

use Company\Model\Repository\ReviewCollection;
use Company\Model\Command\CreateReviewHandler;
use Zend\ServiceManager\ServiceManager;

class CreateReviewHandlerFactory
{
    public function __invoke(ServiceManager $container)
    {
        $reviewCollection = $container->get(ReviewCollection::class);

        $createReviewCommandHandler = new CreateReviewHandler($reviewCollection);
        $createReviewCommandHandler->setServiceLocator($container);

        return $createReviewCommandHandler;
    }
}
```

### Register the **CreateReviewHandler** factory

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    'service_manager' => [
        // rest of code
        'factories' => [
            // Handler
            \Company\Model\Command\CreateReviewHandler::class
            => \Company\Container\Model\CreateReviewHandlerFactory::class,
          ]
          // rest of code
    ],
    // rest of code
];
```

### Link **CreateReview** command to the **CreateReviewHandler**

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    // rest of code
    'prooph' => [
        // rest of code
        'service_bus' => [
            // rest of code
              'command_bus' => [
                'router' => [
                    'routes' => [
                        // rest of code
                        \Company\Model\Command\CreateReview::class
                        => \Company\Model\Command\CreateReviewHandler::class,
                        // rest of code
                ],
            ],
            // rest of code
          ]
    ],
    // rest of code
];
```

### Create **ReviewWasCreated** event

```php
<?php

namespace Company\Model\Event;

use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateChanged;
use User\Model\ValueObject\UserId;

final class ReviewWasCreated extends AggregateChanged
{
    protected $reviewId;
    protected $userId;
    protected $freelancerId;
    protected $comment;

    public static function withData(ReviewId $reviewId, UserId $userId, string $freelancerId, string $comment): ReviewWasCreated
    {
        $event = self::occur(
            $reviewId->toString(), [
                'review_id' => $reviewId->toString(),
                'user_id' => $userId->toString(),
                'freelancer_id' => $freelancerId,
                'comment' => $comment
            ]
        );

        $event->reviewId = $reviewId;
        $event->userId = $userId;
        $event->freelancerId = $freelancerId;
        $event->comment = $comment;

        return $event;
    }

    public function reviewId()
    {
        if (null === $this->reviewId) {
            $this->reviewId = ReviewId::fromString($this->aggregateId());
        }
        
        return $this->reviewId;
    }

    public function userId()
    {
        if (null === $this->userId) {
            $this->userId = UserId::fromString($this->payload['user_id']);
        }
        
        return $this->userId;
    }

    public function freelancerId()
    {
        if (null === $this->freelancerId) {
            $this->freelancerId = $this->payload['freelancer_id'];
        }
        
        return $this->freelancerId;
    }

    public function comment()
    {
        if (null === $this->comment) {
            $this->comment = $this->payload['comment'];
        }
        
        return $this->comment;
    }
}
```

### Register the **ReviewWasCreated** event

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    // rest of code
    'prooph' => [
        'snapshotter' => [
            // rest of code
            'event_names' => [
                // rest of code
                \Company\Model\Event\ReviewWasCreated::class,
            ]
            // rest of code
          ]
    ],
    // rest of code
];
```

### Add **createReviewWithData** method to **Review** aggregate

```php
<?php

namespace Company\Model\Aggregate;

use Company\Model\Event\ReviewWasCreated;
use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateRoot;
use User\Model\ValueObject\UserId;

final class Review extends AggregateRoot
{
    // rest of code
    public static function createReviewWithData(ReviewId $reviewId, UserId $userId, string $freelancerId, string $comment): self
    {
        $review = new self();

        $review->recordThat(ReviewWasCreated::withData(
            $reviewId, $userId, $freelancerId, $comment
        ));

        return $review;
    }
}
```

### Add **whenReviewWasCreated** method to **Review** aggregate

```php
<?php

namespace Company\Model\Aggregate;

use Company\Model\Event\ReviewWasCreated;
use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateRoot;
use User\Model\ValueObject\UserId;

final class Review extends AggregateRoot
{
    // rest of code
    /**
     * @param ReviewWasCreated $event
     * @return void
     */
    protected function whenReviewWasCreated(ReviewWasCreated $event)
    {
        $this->id = $event->reviewId();
        $this->userId = $event->userId();
        $this->freelancerId = $event->freelancerId();
        $this->comment = $event->comment();
    }
}
```

### Create ReviewProjector

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

### Create ReviewProjectorFactory

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

### Register the ReviewProjector factory

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

### Create ReviewProcessManager

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

### Create ReviewProcessManagerFactory

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

### Register ReviewProcessManager factory

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

### Add onReviewWasCreated to ReviewProcessManager

```php
<?php

namespace Company\ProcessManager;

use Company\Projection\ReviewProjector;
use Company\Model\Event\ReviewWasCreated;

class ReviewProcessManager
{
    // rest of code
    /**
     * @param ReviewWasCreated $event
     * @return void
     */
    public function onReviewWasCreated(ReviewWasCreated $event)
    {
        $data = [
            self::ID            => $event->reviewId()->toString(),
            self::USER_ID       => $event->userId()->toString(),
            self::FREELANCER_ID => $event->freelancerId(),
            self::COMMENT       => $event->comment(),
        ];

        $this->reviewProjector->insert($data);
    }
}
```

### Link Event ReviewWasCreated to ReviewProcessManager

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    // rest of code
    'prooph' => [
        'service_bus' => [
            // rest of code
            'event_bus' => [
                'router' => [
                    'routes' => [
                        \Company\Model\Event\ReviewWasCreated::class => [
                            \Company\ProcessManager\ReviewProcessManager::class,
                        ],
                    ]
                ]
            ]
            // rest of code
          ]
    ],
    // rest of code
];
```

### Add **createReviewWithData** and **whenReviewWasCreated** method to **Review** aggregate

```php
<?php

namespace Company\Model\Aggregate;

use Company\Model\Event\ReviewWasCreated;
use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateRoot;
use User\Model\ValueObject\UserId;

final class Review extends AggregateRoot
{
    // rest of code
    /**
     * @param ReviewId $reviewId
     * @param UserId $userId
     * @param UserId $freelancerId
     * @param string $comment
     * @return static
     */
    public static function createReviewWithData(ReviewId $reviewId, UserId $userId, UserId $freelancerId, CompanyId $companyId, string $comment): self
    {
        $review = new self();

        $review->recordThat(ReviewWasCreated::withData(
            $reviewId, $userId, $freelancerId, $companyId, $comment
        ));

        return $review;
    }
    
    protected function whenReviewWasCreated(ReviewWasCreated $event)
    {
        $this->id = $event->reviewId();
        $this->userId = $event->userId();
        $this->freelancerId = $event->freelancerId();
        $this->comment = $event->comment();
    }
}
```

### Add **OnReviewWasCreated** method to **ReviewProcessManager**

```php
<?php

class ReviewProcessManager
{
    // rest of code
    /**
     * @param ReviewWasCreated $event
     * @return void
     */
    public function onReviewWasCreated(ReviewWasCreated $event)
    {
        $data = [
            self::ID            => $event->reviewId()->toString(),
            self::USER_ID       => $event->userId()->toString(),
            self::FREELANCER_ID => $event->freelancerId(),
            self::COMMENT       => $event->comment(),
        ];

        $this->reviewProjector->insert($data);
    }
}
  
```

### Create create\_review.php filter

```php
<?php

return [
    [
        'name' => 'user_id',
        'required' => true,
        'allow_empty' => false,
        'continue_if_empty' => false,
    ],
    [
        'name' => 'freelancer_id',
        'required' => true,
        'allow_empty' => false,
        'continue_if_empty' => false,
    ],
    [
        'name' => 'comment',
        'required' => true,
        'allow_empty' => false,
        'continue_if_empty' => false,
    ]
];
```

### Add CREATE\_REVIEW\_FILTER to the FilterService class

```php
<?php

class FilterService {
    // rest of code
    const CREATE_REVIEW_FILTER = 'create_review';
}
```

### Register **create\_review** filter

```php
<?php

return [
    'input_filter_specs' => [
        // rest of code
        \Company\Validation\FilterService::CREATE_REVIEW_FILTER
        => include __DIR__ . '/filter/create_review.php',
    ],
];
```

Edit The API
------------

### Edit CreateReviewResource

```php
<?php
namespace CompanyAPI\V1\Rest\CreateReview;

use Common\Application\Base\Traits\HireMeTrait;
use Common\Application\Validation\Exception\ValidationException as ValidationExceptionAlias;
use Company\Model\Command\CreateReview;
use Company\Validation\FilterService;
use ZF\ApiProblem\ApiProblem;
use Common\Application\Base\Controller\AbstractResourceListener;

class CreateReviewResource extends AbstractResourceListener
{
    use HireMeTrait;

    /**
     * @param mixed $data
     * @throws ValidationExceptionAlias
     */
    public function create($data)
    {
        $currentUser = $this->getUserBaseOnAccessToken();
        $data->user_id = $currentUser['_id'];

        $command = $this->getCommand(
            CreateReview::class,
            ['payload' => $this->validate(FilterService::CREATE_REVIEW_FILTER, $data)]
        );

        $this->commandBus->dispatch($command);

        return $this->acceptQuery();
    }
}
```

### Edit CreateReviewResourceFactory

```php
<?php
namespace CompanyAPI\V1\Rest\CreateReview;

use Zend\ServiceManager\ServiceLocatorInterface;

class CreateReviewResourceFactory
{
    public function __invoke(ServiceLocatorInterface $services)
    {
        $createReviewResource = new CreateReviewResource();
        $createReviewResource->setServiceLocator($services);

        return $createReviewResource;
    }
}
```
