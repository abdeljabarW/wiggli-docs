# Adding A Push Notification

### Create {NotificationName}Notification

```php
<?php

namespace Notification\Notification\Gateway\Notification;

use Common\Application\Base\Traits\SendNotificationTrait;
use Company\Projection\ReviewFinder;
use Notification\Helper\Service\NotificationService;
use Notification\Model\Event\CreateReviewNotificationDataWasPrepared;
use Notification\Projection\NotificationFinder;
use Notification\Projection\NotificationProjector;
use User\Projection\UserFinder;
use function React\Promise\all;

class SendCreateReviewNotification
{
    use SendNotificationTrait;

    const NOTIFICATION_NAME = 'create_review_notification';
    const MESSAGE = 'A new review was written: "%s"';
    const REVIEW_ID = 'review_id';
    /**
     * @var NotificationService
     */
    private $notificationService;
    /**
     * @var NotificationProjector
     */
    private $notificationProjector;
    /**
     * @var NotificationFinder
     */
    private $notificationFinder;
    /**
     * @var UserFinder
     */
    private $userFinder;
    /**
     * @var ReviewFinder
     */
    private $reviewFinder;

    public function __construct(NotificationService $notificationService, NotificationProjector $notificationProjector, NotificationFinder $notificationFinder, UserFinder $userFinder, ReviewFinder $reviewFinder)
    {
        $this->notificationService = $notificationService;
        $this->notificationProjector = $notificationProjector;
        $this->notificationFinder = $notificationFinder;
        $this->userFinder = $userFinder;
        $this->reviewFinder = $reviewFinder;
    }

    public function __invoke(CreateReviewNotificationDataWasPrepared $event)
    {
        $promise = all(
            $notification = $this->notificationFinder->findById( $event->notificationId()->toString() )
        );

        $promise->then(function() use ($event, $notification) {
            $data = [];
            $review = $this->reviewFinder->findById($reviewId = $event->context()[self::REVIEW_ID]);

            $data[NotificationService::MESSAGE] = [
                NotificationService::TYPE => NotificationService::SYSTEM,
                NotificationService::CONTENT => sprintf(self::MESSAGE, $review['comment']),
                NotificationService::URL => NotificationService::SYSTEM
            ];
            $data[NotificationService::EVENT] = self::NOTIFICATION_NAME;
            $data[NotificationService::CHANNEL] = NotificationService::generateChannelName(
                $userId = $event->userId()->toString()
            );

            $this->saveNotificationContent($event->notificationId()->toString(), $data[NotificationService::MESSAGE]);
            $data[NotificationService::MESSAGE] = array_merge($data[NotificationService::MESSAGE], $notification);

            if ($this->userFinder->isShowNotificationInRealTimeByUserId($userId)) {
                $this->notificationService->pushNotification($data, $userId);
            }
        });
    }
}
```

### Create {NotificationName}NotificationFactory

```php
<?php

namespace Notification\Container\Notification\Gateway\Notification;

use Company\Projection\ReviewFinder;
use Notification\Helper\Service\NotificationService;
use Notification\Notification\Gateway\Notification\SendCreateReviewNotification;
use Notification\Projection\NotificationFinder;
use Notification\Projection\NotificationProjector;
use User\Projection\UserFinder;
use Zend\ServiceManager\ServiceManager;

class SendCreateReviewNotificationFactory
{
    public function __invoke(ServiceManager $serviceManager)
    {
        $notificationService = $serviceManager->get(NotificationService::class);
        $notificationProjector = $serviceManager->get(NotificationProjector::class);
        $userFinder = $serviceManager->get(UserFinder::class);
        $notificationFinder = $serviceManager->get(NotificationFinder::class);
        $reviewFinder = $serviceManager->get(ReviewFinder::class);

        return new SendCreateReviewNotification($notificationService, $notificationProjector, $notificationFinder, $userFinder, $reviewFinder);
    }
}
```

### Register {NotificationName}NotificationFactory

```php
<?php

use Notification\Model\Command\ManageDynamicContentHandler;
use Notification\Model\Event\ReportingSettingsWasManaged;

return [
    'service_manager' => [
        // rest of code
        'factories' => [
            \Notification\Notification\Gateway\Notification\SendCreateReviewNotification::class
            => \Notification\Container\Notification\Gateway\Notification\SendCreateReviewNotificationFactory::class,
          ]
          // rest of code
    ],
    // rest of code
];
```

### Create {NotificationName}NotificationWasPrepared event

```php
<?php

namespace Notification\Model\Event;

class CreateReviewNotificationDataWasPrepared extends AbstractNotificationDataWasPrepared
{

}
```

### Add on{NotificationName}NotificationDataWasPrepared method to NotificationProjector

```php
<?php

namespace Notification\Projection;

// rest of code
final class NotificationProjector
{
    // rest of code
    /**
     * @param CreateReviewNotificationDataWasPrepared $event
     */
    public function onCreateReviewNotificationDataWasPrepared(CreateReviewNotificationDataWasPrepared $event)
    {

        $data = [
            '_id' => $event->notificationId()->toString(),
            'user_recipient' => $this->userFinder->prepareNotificationData($event->userId()->toString()),
            'status' => $event->notificationStatus()->toString(),
            'type' => $event->type(),
            'created_at' => MongoDateUtc::bsonTypeDateTime(time())
        ];

        $this->insert($data);
    }
}
```

### Add when{NotificationName}NotificationDataWasPrepared to Notification aggregate

```php
<?php

namespace Notification\Model\Aggregate;

// rest of code
class Notification extends AggregateRoot {
    // rest of code
    /**
     * @param CreateReviewNotificationDataWasPrepared $event
     */
    public function whenCreateReviewNotificationDataWasPrepared(CreateReviewNotificationDataWasPrepared $event)
    {
        $this->id = $event->notificationId();
        $this->userId = $event->userId();
        $this->notificationStatus = $event->notificationStatus();
        $this->type = $event->type();
        $this->context = $event->context();
    }
}
```

### Register the {NotificationName}NotificationWasPrepared event

```php
<?php

use Notification\Model\Command\ManageDynamicContentHandler;
use Notification\Model\Event\ReportingSettingsWasManaged;

return [
    // rest of code
    'prooph' => [
        'snapshotter' => [
            // rest of code
            'event_names' => [
                // rest of code
                \Notification\Model\Event\CreateReviewNotificationWasPrepared::class,
            ]
            // rest of code
          ]
    ],
    // rest of code
];
```

### Link the {NotificationName}NotificationWasPrepared event to NotificationProjector and SendCreateReviewNotification

```php
<?php

use Notification\Model\Command\ManageDynamicContentHandler;
use Notification\Model\Event\ReportingSettingsWasManaged;

return [
    // rest of code
    'prooph' => [
        'snapshotter' => [
            // rest of code
            'event_bus' => [
                'router' => [
                    'routes' => [
                       \Notification\Model\Event\CreateReviewNotificationDataWasPrepared::class => [
                            \Notification\Projection\NotificationProjector::class,
                            \Notification\Notification\Gateway\Notification\
                            SendCreateReviewNotification::class,
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