CQRS-ES - Resource Update Example
--------

## The Command

### Create The Command

```php
<?php

namespace Company\Model\Command;

use Prooph\Common\Messaging\Command;
use Prooph\Common\Messaging\PayloadConstructable;
use Prooph\Common\Messaging\PayloadTrait;
use Company\Model\ValueObject\ReviewId;

final class UpdateReview extends Command implements PayloadConstructable
{
    use PayloadTrait;

    public function withData(string $reviewId, string $comment)
    {
        return new self([
            'id' => $reviewId,
            'comment' => $comment,
        ]);
    }

    /**
     * @return ReviewId
     */
    public function reviewId(): ReviewId
    {
        return ReviewId::fromString($this->payload['id']);
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
use Common\Rbac\Identity\IdentityTrait;
use Company\Company\Model\Exception\ReviewException;
use Company\Model\Aggregate\Review;
use Company\Model\Repository\ReviewCollection;
use Company\Projection\ReviewFinder;
use User\Projection\UserFinder;

class UpdateReviewHandler
{
    use IdentityTrait;
    use HireMeTrait;

    /**
     * @var ReviewCollection
     */
    private $reviewCollection;

    public function __construct(ReviewCollection $reviewCollection)
    {
        $this->reviewCollection = $reviewCollection;
    }

    public function __invoke(UpdateReview $updateReview)
    {
        /** @var Review $review */
        if (!$review = $this->reviewCollection->get($reviewId = $updateReview->reviewId())) {
            throw ReviewException::withReviewId();
        }

        $review->updateReviewWithData($reviewId, $updateReview->comment());
    }
}
```

### Create The Command Handler Factory

```php
<?php

namespace Company\Container\Model;

use Company\Model\Command\UpdateReviewHandler;
use Company\Model\Repository\ReviewCollection;
use Zend\ServiceManager\ServiceManager;

class UpdateReviewHandlerFactory
{
    public function __invoke(ServiceManager $container)
    {
        $reviewCollection = $container->get(ReviewCollection::class);

        $updateReviewCommandHandler = new UpdateReviewHandler($reviewCollection);
        $updateReviewCommandHandler->setServiceLocator($container);

        return $updateReviewCommandHandler;
    }
}
```

### Register Command Handler Factory

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    'service_manager' => [
        // rest of code
        'factories' => [
            // Handler
            \Company\Model\Command\UpdateReviewHandler::class
            => \Company\Container\Model\UpdateReviewHandlerFactory::class,
          // rest of code
    ],
    // rest of code
];
```

### Link The Command To The Command Handler

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
                        \Company\Model\Command\UpdateReview::class
                        => \Company\Model\Command\UpdateReviewHandler::class,
                        // rest of code
                ],
            ],
            // rest of code
          ]
    ],
    // rest of code
];
```

## The Event

### Create The Event

```php
<?php

namespace Company\Model\Event;

use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateChanged;

class ReviewWasUpdated extends AggregateChanged
{
    protected $reviewId;
    protected $comment;

    public static function withData(ReviewId $reviewId,  string $comment)
    {
        $event = self::occur(
            $reviewId->toString(), [
                'comment' => $comment,
            ]
        );

        $event->reviewId = $reviewId;
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
                \Company\Model\Event\ReviewWasUpdated::class,
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
        'snapshotter' => [
            // rest of code
            'event_bus' => [
                'router' => [
                    'routes' => [
                        \Company\Model\Event\ReviewWasUpdated::class => [
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

## Dispatching The Event On The Aggregate And {AggregateName}ProcessManager

### Add {commandName}WithData Method To Aggregate

```php
<?php

namespace Company\Model\Aggregate;

use Company\Model\Event\ReviewWasCreated;
use Company\Model\Event\ReviewWasDeleted;
use Company\Model\Event\ReviewWasUpdated;
use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateRoot;
use User\Model\ValueObject\UserId;

final class Review extends AggregateRoot
{
    // rest of code
    public function updateReviewWithData(ReviewId $reviewId, string $comment)
    {
        $this->recordThat(ReviewWasUpdated::withData(
            $reviewId, $comment
        ));
    }
}
```

### Add on{EventName} method to {AggregateName}ProcessManager

```php
<?php

namespace Company\Model\Aggregate;

use Company\Model\Event\ReviewWasCreated;
use Company\Model\Event\ReviewWasDeleted;
use Company\Model\Event\ReviewWasUpdated;
use Company\Model\ValueObject\ReviewId;
use Prooph\EventSourcing\AggregateRoot;
use User\Model\ValueObject\UserId;

final class Review extends AggregateRoot
{
    // rest of code
    public function onReviewWasUpdated(ReviewWasCreated $event)
    {
        $this->reviewProjector->update(
            [self::ID => $event->reviewId()->toString()],
            [
                '$set' => [
                    self::COMMENT => $event->comment(),
                ]
            ]
        );
    }
}
```

### Add when{EventName} Method To The Aggregate

```php
<?php

namespace Company\Container\ProcessManager;

use Company\ProcessManager\ReviewProcessManager;
use Company\Projection\ReviewProjector;
use Zend\ServiceManager\ServiceManager;

class ReviewProcessManager
{
    // rest of code
    protected function whenReviewWasUpdated(ReviewWasUpdated $event)
    {
        $this->id = $event->reviewId();
        $this->comment = $event->comment();
    }
}
```

## Add The Command Dispatch To The Api

### Create The Filter File

```php
<?php

return [
    [
        'name' => 'id',
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

### Add {FILTER_NAME} Constant To The FilterService class

```php
<?php

namespace Company\Validation;

class FilterService
{
  // rest of code
  const EDIT_REVIEW_FILTER = 'edit_review';
}
```

### Save The Filter File With The FILTER_NAME As A Key

```php
<?php

return [
    'input_filter_specs' => [
      // rest of code
      \Company\Validation\FilterService::EDIT_REVIEW_FILTER
        => include __DIR__ . '/filter/edit_review.php',
      ]
];
```

### Modify The Api Resource 

```php
namespace CompanyAPI\V1\Rest\UpdateReview;

use Common\Application\Base\Controller\AbstractResourceListener;
use Common\Application\Base\Traits\HireMeTrait;
use Common\Application\Validation\Exception\ValidationException;
use Company\Model\Command\UpdateReview;
use Company\Validation\FilterService;

class UpdateReviewResource extends AbstractResourceListener
{
    use HireMeTrait;

    /**
     * Update a resource
     *
     * @param mixed $id
     * @param mixed $data
     * @throws ValidationException
     */
    public function update($id, $data)
    {
        $currentUser = $this->getUserBaseOnAccessToken();
        $data->user_id = $currentUser['_id'];
        $data->id = $id;

        $command = $this->getCommand(UpdateReview::class,
            ['payload' => $this->validate(FilterService::EDIT_REVIEW_FILTER, $data)]
        );

        $this->commandBus->dispatch($command);

        return $this->acceptQuery();
    }
}
```