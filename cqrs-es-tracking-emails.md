Tracking An Email
----

- [Create Command Track{EmailName}Email](#create-command-track-emailname-email)
- [Create Command Handler Track{EmailName}EmailHandler](#create-command-handler-track-emailname-emailhandler)
- [Create Track{EmailName}EmailHandlerFactory](#create-track-emailname-emailhandlerfactory)
- [Register Track{EmailName}EmailHandlerFactory](#register-track-emailname-emailhandlerfactory)
- [Link Track{EmailName}Email command to TrackCreateReviewEmailHandler](#link-track-emailname-email-command-to-trackcreatereviewemailhandler)
- [Create Event {EmailName}EmailWasSent](#create-event--emailname-emailwassent)
- [Register Event {EmailName}EmailWasSent](#register-event--emailname-emailwassent)
- [Link Event {EmailName}EmailWasSent with EmailProjector](#link-event--emailname-emailwassent-with-emailprojector)
- [Link Event {EmailName}EmailWasSent with ReviewProcessManager](#link-event--emailname-emailwassent-with-reviewprocessmanager)
- [Add on{EmailName}EmailWasSent to ReviewProcessManager](#add-on-emailname-emailwassent-to-reviewprocessmanager)
- [Add track{EmailName}WithData method to Aggregate Review](#add-track-emailname-withdata-method-to-aggregate-review)
- [Add when{EmailName}EmailWasSent method to Aggregate Review](#add-when-emailname-emailwassent-method-to-aggregate-review)
- [Register create_review.html template](#register-create-reviewhtml-template)
- [Register Send{EmailName}Email Notification into EmailType Helper](#register-send-emailname-email-notification-into-emailtype-helper)

## Create Command Track{EmailName}Email

```php
<?php

namespace Email\Model\Command;

use Company\Model\ValueObject\ReviewId;
use Email\Model\Event\AbstractEmailWasSent;
use Email\Model\Event\CreateReviewEmailWasSent;
use Email\Model\ValueObject\EmailId;
use Email\Model\ValueObject\EmailStatus;
use Prooph\Common\Messaging\Command;
use Prooph\Common\Messaging\PayloadConstructable;
use Prooph\Common\Messaging\PayloadTrait;
use User\Model\ValueObject\UserId;

class TrackCreateReviewEmail extends Command implements PayloadConstructable
{
    use PayloadTrait;

    /**
     * @param string $userId
     * @param string $emailId
     * @param string $emailStatus
     * @param string $reviewId
     * @return TrackCreateReviewEmail
     */
    public static function withData(string $userId, string $emailId, string $emailStatus, string $reviewId): TrackCreateReviewEmail
    {
        return new self([
            'id' => $userId,
            'email_id' => $emailId,
            'email_status' => $emailStatus,
            'review_id' => $reviewId,
        ]);
    }

    /**
    * @return UserId
    */
    public function userId()
    {
        return UserId::fromString($this->payload['id']);
    }

    /**
     * @return EmailId
     */
    public function emailId()
    {
        return EmailId::fromString($this->payload[AbstractEmailWasSent::EMAIL_ID]);
    }

    /**
     * @return EmailStatus
     */
    public function emailStatus()
    {
        return EmailStatus::fromString($this->payload[AbstractEmailWasSent::EMAIL_STATUS]);
    }

    /**
     * @return ReviewId
     */
    public function reviewId()
    {
        return ReviewId::fromString($this->payload[CreateReviewEmailWasSent::REVIEW_ID]);
    }
}
```

## Create Command Handler Track{EmailName}EmailHandler

```php
<?php

namespace Email\Container\Model;

use Email\Model\Command\TrackCreateReviewEmailHandler;
use Email\Projection\EmailFinder;
use Company\Model\Repository\ReviewCollection;
use User\Projection\UserFinder;
use Zend\ServiceManager\ServiceManager;

class TrackCreateReviewEmailHandlerFactory
{
    /**
     * @param ServiceManager $container
     * @return TrackCreateReviewEmailHandler
     */
    public function __invoke(ServiceManager $container)
    {
        return new TrackCreateReviewEmailHandler(
            $container->get(UserFinder::class),
            $container->get(EmailFinder::class),
            $container->get(ReviewCollection::class)
        );
    }
}
```

## Create Track{EmailName}EmailHandlerFactory

```php
<?php

namespace Email\Container\Model;

use Email\Model\Command\TrackCreateReviewEmailHandler;
use Email\Projection\EmailFinder;
use Company\Model\Repository\ReviewCollection;
use User\Projection\UserFinder;
use Zend\ServiceManager\ServiceManager;

class TrackCreateReviewEmailHandlerFactory
{
    /**
     * @param ServiceManager $container
     * @return TrackCreateReviewEmailHandler
     */
    public function __invoke(ServiceManager $container)
    {
        return new TrackCreateReviewEmailHandler(
            $container->get(UserFinder::class),
            $container->get(EmailFinder::class),
            $container->get(ReviewCollection::class)
        );
    }
}
```

## Register Track{EmailName}EmailHandlerFactory

```php
<?php

use Company\Model\Command\ManageDynamicContentHandler;
use Company\Model\Event\ReportingSettingsWasManaged;

return [
    'service_manager' => [
        // rest of code
        'factories' => [
            // Handler
            \Email\Model\Command\TrackCreateReviewEmailHandler::class
            => \Email\Container\Model\TrackCreateReviewEmailHandlerFactory::class,
            // rest of code
    ],
    // rest of code
];
```

## Link Track{EmailName}Email command to TrackCreateReviewEmailHandler

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
                        \Email\Model\Command\TrackCreateReviewEmail::class
                        => \Email\Model\Command\TrackCreateReviewEmailHandler::class,
                        // rest of code
                ],
            ],
            // rest of code
          ]
    ],
    // rest of code
];
```

## Create Event {EmailName}EmailWasSent

```php
<?php

namespace Email\Model\Event;

use Company\Model\ValueObject\ReviewId;
use Email\Model\ValueObject\EmailId;
use Email\Model\ValueObject\EmailStatus;
use User\Model\ValueObject\UserId;

class CreateReviewEmailWasSent extends AbstractEmailWasSent
{
    public const REVIEW_ID = 'review_id';

    private $reviewId;

    public static function withData(
        UserId $userId,
        EmailId $emailId,
        EmailStatus $emailStatus,
        ReviewId $reviewId
    ) {
        $event = self::occur(
            $userId->toString(),
            [
                self::USER_ID => $userId->toString(),
                self::EMAIL_ID => $emailId->toString(),
                self::EMAIL_STATUS => $emailStatus->toString(),
                self::REVIEW_ID => $reviewId->toString(),
            ]
        );

        $event->userId = $userId;
        $event->emailId = $emailId;
        $event->emailStatus = $emailStatus;
        $event->reviewId = $reviewId;

        return $event;
    }

    /**
     * @return ReviewId
     */
    public function reviewId()
    {
        if (null === $this->reviewId) {
            $this->reviewId = ReviewId::fromString($this->payload[self::REVIEW_ID]);
        }

        return $this->reviewId;
    }
}
```

## Register Event {EmailName}EmailWasSent

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
                \Email\Model\Event\CreateReviewEmailWasSent::class
            ]
            // rest of code
          ]
    ],
    // rest of code
];
```

## Link Event {EmailName}EmailWasSent with EmailProjector

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
                        \Email\Model\Event\CreateReviewEmailWasSent::class => [
                            \Email\Projection\EmailProjector::class,
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

## Link Event {EmailName}EmailWasSent with ReviewProcessManager

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
                        \Email\Model\Event\CreateReviewEmailWasSent::class => [
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

## Add on{EmailName}EmailWasSent to ReviewProcessManager

```php
<?php

namespace Company\ProcessManager;

// rest of code

class ReviewProcessManager
{
    // rest of code
    /**
     * @param CreateReviewEmailWasSent $event
     * @return void
     */
    public function onCreateReviewEmailWasSent(CreateReviewEmailWasSent $event)
    {
        $this->emailProjector->update(
            [self::ID => $event->emailId()->toString()],
            ['$set' => [self::STATUS => $event->emailStatus()->toString()]]
        );
    }
    // rest of code
}
```

## Add track{EmailName}WithData method to Aggregate Review

```php
<?php

namespace Company\Model\Aggregate;

// rest of code
final class Review extends AggregateRoot
{
    // rest of code
    /**
     * @param ReviewId $reviewId
     * @param EmailId $emailId
     * @param EmailStatus $emailStatus
     * @param UserId $userId
     * @return void
     */
    public function trackCreateReviewWithData(
        UserId $userId,
        EmailId $emailId,
        EmailStatus $emailStatus,
        ReviewId $reviewId
    ) {
        $this->recordThat(
            CreateReviewEmailWasSent::withData(
                $userId,
                $emailId,
                $emailStatus,
                $reviewId
            )
        );
    }
}
```

## Add when{EmailName}EmailWasSent method to Aggregate Review

```php
<?php

namespace Company\Model\Aggregate;

// rest of code

final class Review extends AggregateRoot
{
    // rest of code
    /**
     * @param CreateReviewEmailWasSent $event
     * @return void
     */
    public function whenCreateReviewEmailWasSent(CreateReviewEmailWasSent $event)
    {
        $this->id = $event->reviewId();
        $this->userId = $event->userId();
        $this->emailId = $event->emailId();
        $this->emailStatus = $event->emailStatus();
    }
}
```

## Register create_review.html template

```php
<?php

namespace {
    // rest of code
    const INTERVIEW_EMAIL_TEMPLATE = [
        // rest of code
        'create_review.html'
    ];
    // rest of code
}
```

## Register Send{EmailName}Email Notification into EmailType Helper

```php
<?php

namespace Email\Helper;

// rest of code
class EmailType
{
    // rest of code
    const CREATE_REVIEW_EMAIL = SendCreateReviewEmail::EMAIL_NAME;

    const EMAIL_COMMANDS = [
        // rest of code
        self::CREATE_REVIEW_EMAIL => TrackCreateReviewEmail::class,
    ];
}
```