# CQRS
## 1. The Command

### Create The Command

```php
<?php

namespace Company\Model\Command;

use Company\Model\ValueObject\ReviewId;
use Prooph\Common\Messaging\Command;
use Prooph\Common\Messaging\PayloadConstructable;
use Prooph\Common\Messaging\PayloadTrait;

final class DeleteReview extends Command implements PayloadConstructable
{
    use PayloadTrait;

    public static function withData(string $id)
    {
        return new self(
            [
                'id' => $id,
            ]
        );
    }

    /**
     * @return ReviewId
     */
    public function reviewId()
    {
        return ReviewId::fromString($this->payload['id']);
    }
}
```

### Create The Command Handler

```php
<?php

namespace Company\Model\Command;

use Common\Application\Base\Traits\ManageAccountDeletionTrait;
use Common\Rbac\Identity\IdentityTrait;
use Company\Company\Model\Exception\ReviewException;
use Company\Model\Aggregate\Review;
use Company\Model\Repository\ReviewCollection;
use Company\Projection\ReviewFinder;
use Zend\ServiceManager\ServiceLocatorInterface;

class DeleteReviewHandler
{
    use IdentityTrait;
    use ManageAccountDeletionTrait;

    /**
     * @var ReviewFinder
     */
    private $reviewFinder;
    /**
     * @var ReviewCollection
     */
    private $reviewCollection;
    /**
     * @var ServiceLocatorInterface
     */
    private $serviceLocator;

    public function __construct(ReviewFinder $reviewFinder, ReviewCollection $reviewCollection, ServiceLocatorInterface $serviceLocator)
    {
        $this->reviewFinder = $reviewFinder;
        $this->reviewCollection = $reviewCollection;
        $this->serviceLocator = $serviceLocator;
    }

    /**
     * @param DeleteReview $deleteReview
     * @return void
     */
    public function __invoke(DeleteReview $deleteReview)
    {
        /** @var Review $review */
        if (!$review = $this->reviewCollection->get($reviewId = $deleteReview->reviewId())) {
            throw ReviewException::withReviewId();
        }

        $this->deleteReviewWithData($review->aggregateId());
    }
}
```

### Create Command Handler Factory

```php
<?php

namespace Company\Container\Model;

use Company\Model\Command\DeleteReviewHandler;
use Company\Model\Repository\ReviewCollection;
use Company\Projection\ReviewFinder;
use Zend\ServiceManager\ServiceManager;

final class DeleteReviewHandlerFactory
{
    /**
     * @param ServiceManager $container
     * @return DeleteReviewHandler
     */
    public function __invoke(ServiceManager $container)
    {
        return new DeleteReviewHandler(
            $container->get(ReviewFinder::class),
            $container->get(ReviewCollection::class),
            $container
        );
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
            \Company\Model\Command\DeleteReviewHandler::class
            => \Company\Container\Model\DeleteReviewHandlerFactory::class,
          // rest of code
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

class ReviewWasDeleted extends AggregateChanged
{
    private $reviewId;

    public static function withData(ReviewId $reviewId)
    {
        $event = self::occur($reviewId->toString(), []);
        $event->reviewId = $reviewId;

        return $event;
    }

    /**
     * @return ReviewId
     */
    public function reviewId()
    {
        if (null === $this->reviewId) {
            $this->reviewId = ReviewId::fromString($this->aggregateId());
        }

        return $this->reviewId;
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
                \Company\Model\Event\ReviewWasDeleted::class,
            ]
            // rest of code
          ]
    ],
    // rest of code
];
```

### Link The Event To {Aggregate}ProcessManager

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
                        \Company\Model\Event\ReviewWasDeleted::class => [
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

### Add {commandName}WithData To The Aggregate

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
    public function deleteReviewWithData(ReviewId $reviewId)
    {
        $this->recordThat(ReviewWasDeleted::withData($reviewId));
    }
}
```

### Add On{EventName} Method To {Aggregate}ProcessManager

```php
<?php

namespace Company\ProcessManager;

use Company\Model\Event\ReviewWasDeleted;
use Company\Model\Event\ReviewWasUpdated;
use Company\Projection\ReviewProjector;
use Company\Model\Event\ReviewWasCreated;

class ReviewProcessManager
{
    // rest of code
    public function onReviewWasDeleted(ReviewWasDeleted $event)
    {
        $this->reviewProjector->remove(['_id' => $event->reviewId()->toString()]);
    }
}
```

### Add when{EventName} Method To The Aggregate

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
    protected function whenReviewWasDeleted(ReviewWasDeleted $event)
    {
      $this->id = $event->reviewId();
    }
}
```

## 4. Add The Command Dispatch To The Api

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
];
```

### Add {FILTER_NAME}_FILTER constant to the FilterService class

```php
<?php

namespace Company\Validation;

class FilterService
{
  // rest of code
  const DELETE_REVIEW_FILTER = 'delete_review';
```

### Save The Filter File With The FILTER_NAME As A Key

```php
<?php

return [
    'input_filter_specs' => [
      // rest of code
      \Company\Validation\FilterService::DELETE_REVIEW_FILTER
        => include __DIR__ . '/filter/delete_review.php',
      ]
];
```

### Modify The Api Resource

```php
<?php
namespace CompanyAPI\V1\Rest\DeleteReview;

use Common\Application\Base\Controller\AbstractResourceListener;
use Common\Application\Base\Traits\HireMeTrait;
use Company\Model\Command\DeleteReview;
use Company\Validation\FilterService;
use stdClass;
use ZF\ApiProblem\ApiProblem;

class DeleteReviewResource extends AbstractResourceListener
{
    use HireMeTrait;

    /**
     * Delete a resource
     *
     * @param mixed $id
     * @return ApiProblem|mixed
     * @throws \Common\Application\Validation\Exception\ValidationException
     */
    public function delete($id)
    {
        $data = new StdClass();
        $data->id = $id;

        $command = $this->getCommand(DeleteReview::class, [
            'payload' => $this->validate(FilterService::DELETE_REVIEW_FILTER, $data)]
        );

        $this->commandBus->dispatch($command);

        return $this->acceptQuery();
    }
}
```