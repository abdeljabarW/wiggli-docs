CQRS-ES - Resource Creation Example
-------

## 1. The Command

### Create The Command

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

### Create The Command Handler

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

### Create The Command Handler Factory

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

### Register The Command Handler Factory

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

### Link The Command To The Handler

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

## 2. The Event

### Create The Event

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

### Register The Event

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

### Link The Event To {AggregateName}ProcessManager

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

## 3. Dispatching The Event On The Aggregate And {AggregateName}ProcessManager

### Add {commandName}WithData Method To The Aggregate

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

### Add on{EventName} to {AggregateName}ProcessManager

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

### Add when{EventName} Method To The Aggregate

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

## 4. Add The Command Dispatch To The Api

### Create The Filter File

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

### Add {FILTER_NAME}_FILTER constant to the FilterService class

```php
<?php

class FilterService {
    // rest of code
    const CREATE_REVIEW_FILTER = 'create_review';
}
```

### Register The New Filter File

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

### Modify The Api Resource

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
