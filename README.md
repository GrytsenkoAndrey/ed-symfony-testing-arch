# ed-symfony-testing-arch
Testing Application Architecture with Pest. [Official documentation](https://pestphp.com/docs/arch-testing)

The health and scalability of a project, as well as its ease of maintenance, heavily rely on the quality of its architecture.
This is particularly crucial when collaborating with extensive teams on projects, ensuring adherence to standards and minimizing unexpected challenges.

I recently discovered Pest php’s capabilities to test projects architecture,
In this article, we will explore together some applications for this amazing feature.

## Installation
```
composer require pestphp/pest --dev --with-all-dependencies
```

Without losing time, we will jump to real life applications.

## ✅Do not forget Dumps in production

Forgetting debugging statements or dumps in our code base, is so common, it happened to all of us 😅.
using pest php we can eliminate this possibility once for all.

```
<?php

arch('Do not forget dumps in your production code')
    ->expect(['dd', 'dump', 'exit', 'die', 'print_r', 'var_dump', 'echo', 'print'])
    ->not
    ->toBeUsed();
```

## ✅Validating Controllers

Consider the following rules:

- Controllers must extends AbstractController ( Official Best Practice in Symfony)
- Controllers must has “Controller” as suffix
- Controller cannot be used in other classes
**Test cases**

```
<?php

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

arch('controllers must has "Controller" as suffix')
    ->expect('App\Controller')
    ->toHaveSuffix('Controller');

arch(sprintf('constroller must extends %s', AbstractController::class))
    ->expect('App\Controller')
    ->toExtend(AbstractController::class);

arch('Controllers cannot be used anywhere')
    ->expect('App\Controller')
    ->toBeUsedInNothing();
```


## ✅Validating Handlers

I personally love using handlers to properly handle http requests, commands, and uses cases in general.
Supposed we want all of our handlers to use the same interface.

```
<?php

namespace App\Handler;

use App\Payload\PayloadInterface;

interface HandlerInterface
{
    public function handle(PayloadInterface $payload);
}
```

**Rules**
We can check our handlers against these rules:

- All handlers must final classes
- All class in handlers package/namespace must implement our handlers interface

```
<?php

use App\Handler\CreateOrUpdateUserHandler;
use App\Handler\HandlerInterface;

arch(sprintf('CreateOrUpdateUserHandler must implement %s', HandlerInterface::class))
->expect(CreateOrUpdateUserHandler::class)
->toImplement(HandlerInterface::class);

arch('handlers must be final')
    ->expect('App\Handler')
    ->classes
    ->toBeFinal();

arch(sprintf('All classes from Namespace App\Handler implement %s', HandlerInterface::class))
    ->expect('App\Handler')
    ->toImplement(HandlerInterface::class);
```
  

## ✅Validating Value Objects

Let’s define some custom rules for a special type of Value objects: Request Payloads.

Rules
- Payloads must implements App\Payload\PayloadInterface
- Payload must be readonly, as we do not want our payload value to change across the business process application.
The following Payload is an example for a payload respecting rules above.

```
<?php

namespace App\Payload;

use Symfony\Component\Validator\Constraints as Assert;
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\Type;

readonly class User implements PayloadInterface
{
    public function __construct(
        #[Length(min: 3)]
        #[Type(type: 'alpha')]
        public string $name,
        #[Assert\Valid]
        public ?Address $address = null,
    ) {
    }
}
```

We only need to create tests to ensure that these rules are respected.

```
<?php

use App\Payload\PayloadInterface;

arch('Payloads should be readonly')
    ->expect('App\Payload')
    ->classes()
    ->toBeReadonly();

arch('Payloads must implement ' . PayloadInterface::class)
    ->expect('App\Payload')
    ->classes()
    ->toImplement(PayloadInterface::class);
```

If a payload doesn’t respect this rule, Pest PHP with alert us and fails the test.
![image](https://github.com/GrytsenkoAndrey/ed-symfony-testing-arch/assets/63291871/ce8c62b1-e286-4b77-a8c5-311392af7003)

After fixing the issue all tests must pass
![image](https://github.com/GrytsenkoAndrey/ed-symfony-testing-arch/assets/63291871/ba9239d2-3aeb-4f03-887f-331bb813b8b8)

For more details about payloads usage, I wrote a detailed article explaining [how to use payloads to handle requests.](https://medium.com/@mounir.mouih/how-to-properly-handle-requests-with-symfony-6-3-0bfc8d7726a9)


