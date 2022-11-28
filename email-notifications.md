# Adding An Email Notification

### Create Send{NotificationName}Email

```php
<?php

namespace Email\Notification\Gateway\Email;

use Common\Application\Base\Traits\SendEmailTrait;
use Common\Application\Notification\Gateway\Email;
use Common\Application\Notification\Service\NotificationService;
use Company\Projection\ReviewFinder;
use Email\Model\Event\CreateReviewEmailDataWasPrepared;
use Email\Projection\EmailProjector;
use User\Projection\UserFinder;

final class SendCreateReviewEmail
{
    use SendEmailTrait;

    const EMAIL_NAME = 'create_review';
    const EMAIL_SUBJECT = 'A new review was written.';
    const USER_ID = 'user_id';
    const REVIEW_ID = 'review_id';

    /**
     * @var NotificationService
     */
    private $notificationService;
    /**
     * @var EmailProjector
     */
    private $emailProjector;
    /**
     * @var UserFinder
     */
    private $userFinder;
    /**
     * @var Email
     */
    private $emailMessage;
    /**
     * @var array
     */
    private $emailConfig;
    /**
     * @var ReviewFinder
     */
    private $reviewFinder;

    /**
     * @param NotificationService $notificationService
     * @param Email $emailMessage
     * @param array $emailConfig
     * @param EmailProjector $emailProjector
     * @param UserFinder $userFinder
     * @param ReviewFinder $reviewFinder
     */
    public function __construct(NotificationService $notificationService, Email $emailMessage, array $emailConfig, EmailProjector $emailProjector, UserFinder $userFinder, ReviewFinder $reviewFinder)
    {
        $this->notificationService = $notificationService;
        $this->emailMessage = $emailMessage;
        $this->emailConfig = $emailConfig;
        $this->emailProjector = $emailProjector;
        $this->userFinder = $userFinder;
        $this->reviewFinder = $reviewFinder;
    }

    public function __invoke(CreateReviewEmailDataWasPrepared $event)
    {
        $user = $this->userFinder->findById($event->userId());
        $review = $this->userFinder->findById($event->context()[self::REVIEW_ID]);

        $recipient[] = [
            'name' => $user['first_name'],
            'email' => $user['email'],
        ];

        $this->emailMessage->buildMessage(
            $message = array_merge([
                'user_id' => $event->userId(),
                'email_id' => $emailId = $event->emailId()->toString(),
                'template_name' => self::EMAIL_NAME,
                'subject' => self::EMAIL_SUBJECT,
                'recipient' => array_values($recipient),
                'merge_contents' => [
                    $this->mergeContentVar('first_name', ucfirst($user['first_name'])),
                    $this->mergeContentVar('client_name', 'someone'),
                    $this->mergeContentVar('email_address', $user['email']),
                    $this->mergeContentVar('review', $review['comment']),
                ],
            ],
                $this->emailConfig
            )
        );

        $this->saveMessageContent($emailId, $message);
        $this->notificationService->createNotification($this->emailMessage);
    }
}
```

### Create Send{NotificationName}EmailFactory

```php
<?php

namespace Email\Container\Notification\Gateway\Email;

use Common\Application\Notification\Gateway\Email;
use Common\Application\Notification\Service\NotificationService;
use Company\Projection\ReviewFinder;
use Email\Notification\Gateway\Email\SendCreateReviewEmail;
use Email\Projection\EmailProjector;
use Interop\Config\ConfigurationTrait;
use Interop\Config\RequiresConfig;
use Interop\Config\RequiresMandatoryOptions;
use User\Projection\UserFinder;
use Zend\ServiceManager\ServiceManager;

class SendCreateReviewEmailFactory implements RequiresConfig, RequiresMandatoryOptions
{
    use ConfigurationTrait;

    public function __invoke(ServiceManager $serviceManager)
    {
        $notificationService = $serviceManager->get(NotificationService::class);
        $emailMessage = $serviceManager->get(Email::class);

        $config = $serviceManager->get('config');
        $emailConfig = $this->options($config);

        $emailProjector = $serviceManager->get(EmailProjector::class);
        $userFinder = $serviceManager->get(UserFinder::class);
        $reviewFinder = $serviceManager->get(ReviewFinder::class);

        return new SendCreateReviewEmail(
            $notificationService,
            $emailMessage,
            $emailConfig,
            $emailProjector,
            $userFinder,
            $reviewFinder
        );
    }

    public function dimensions()
    {
        return ['HireMe', 'Config'];
    }

    public function mandatoryOptions()
    {
        return [
            'sender_full_name',
            'sender_email',
            'merge_language',
        ];
    }
}
```

### Register the Send{NotificationName}EmailFactory

```php
<?php

return [
    'service_manager' => [
        // rest of code
        'factories' => [
          \Email\Notification\Gateway\Email\SendCreateReviewEmail::class
          => \Email\Container\Notification\Gateway\Email\SendCreateReviewEmailFactory::class,
        ]
          // rest of code
    ],
    // rest of code
];
```

### Create {NotificationName}EmailDataWasPrepared event

```php
<?php

namespace Email\Model\Event;

use User\Model\ValueObject\UserId;

final class CreateReviewEmailDataWasPrepared extends AbstractExternalEmailDataWasPrepared
{
}
```

### Add on{NotificationName}EmailDataWasPrepared method to EmailProjector

```php
<?php

namespace Email\Projection;

// rest of code
class EmailProjector {
    // rest of code
    /**
     * @param CreateReviewEmailDataWasPrepared $event
     */
     public function onCreateReviewEmailDataWasPrepared(CreateReviewEmailDataWasPrepared $event)
    {
        $data = [
            '_id' => $event->emailId()->toString(),
            'user_id' => $event->userId(),
            'review_id' => $event->context()[SendCreateReviewEmail::REVIEW_ID],
            'email' => $event->email()->toString(),
            'status' => $event->emailStatus()->toString(),
            'template' => $event->template()
        ];

        $this->insert($data);
    }
}
```

### Add when{NotificationName}EmailDataWasPrepared to Email aggregate

```php
<?php

class Email extends AggregateRoot {
    // rest of code
    public function whenCreateReviewEmailDataWasPrepared(CreateReviewEmailDataWasPrepared $event)
    {
        $this->id = $event->emailId();
        $this->userId = $event->userId();
        $this->email = $event->email();
        $this->emailStatus = $event->emailStatus();
        $this->template = $event->template();
        $this->context = $event->context();
    }
}
```

### Add Send{NotificationName}Email::EMAIL_NAME case to Email::prepareEmailDataWithData

```php
<?php

class Email extends AggregateRoot {
    // rest of code
    
    /**
     * @param EmailId $emailId
     * @param UserId $userId
     * @param EmailAddress $email
     * @param EmailStatus $emailStatus
     * @param string $template
     * @param array $context
     * @return Email
     */
    public static function prepareEmailDataWithData(
        EmailId $emailId,
        UserId $userId,
        EmailAddress $email,
        EmailStatus $emailStatus,
        string $template,
        array $context
    ) {
        $self = new self();
        switch ($template) {
            // rest of code
            case SendCreateReviewEmail::EMAIL_NAME:
                $self->recordThat(
                    CreateReviewEmailDataWasPrepared::withData(
                        $emailId,
                        $userId->toString(),
                        $email,
                        $emailStatus,
                        $template,
                        $context
                    )
                );
                break;
        }
        
        return $self;
      }
}
```

### Register {NotificationName}EmailDataWasPrepared event

```php
<?php

return [
    // rest of code
    'prooph' => [
        'snapshotter' => [
            // rest of code
            'event_names' => [
                // rest of code
                \Email\Model\Event\CreateReviewEmailDataWasPrepared::class,
            ]
            // rest of code
          ]
    ],
    // rest of code
];
```

### Link the {NotificationName}EmailWasPrepared event to NotificationProjector and SendCreateReviewEmail

```php
<?php

return [
    // rest of code
    'prooph' => [
        'snapshotter' => [
            // rest of code
            'event_bus' => [
                'router' => [
                    'routes' => [
                       \Email\Model\Event\CreateReviewEmailDataWasPrepared::class => [
                            \Email\Projection\EmailProjector::class,
                            \Email\Notification\Gateway\Email\
                            SendCreateReviewEmail::class,
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

### Create email template

```html
<h1>A new review was written</h1>
<p>Hi *|FIRST_NAME|*, </p>
<p>A new review was written on your profile:</p>
<p>
    Client : *|CLIENT_NAME|*<br/>
    Email : *|EMAIL_ADDRESS|*<br/>
</p>
<p>*|REVIEW|*</p>
<p>
    Have a nice day!
</p>
```