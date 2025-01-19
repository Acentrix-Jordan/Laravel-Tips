# Container Tips ([cd ..](./README.md))

- [Use rebinding events to refresh dependencies](#laravel-tip--use-rebinding-events-to-refresh-dependencies-️)
- [Bind Typed Variadics](#laravel-tip--bind-typed-variadics-️)
- [Check Your Application Enviroment](#laravel-tip--check-your-application-enviroment-️)
- [The New "optimizes" Method](#laravel-tip--the-new-optimizes-method-️)
- [Deferred Providers](#laravel-tip--deferred-providers-️)

## Laravel Tip 💡: Use rebinding events to refresh dependencies ([⬆️](#container-tips-cd-))

When a bound instance in the Laravel container is rebound, a rebinding event is triggered. You can listen to this event to ensure that the classes using the instance stay up to date. You can achieve this by using the rebinding method or simply using the refresh shortcut 🚀

```php
<?php

namespace App\Providers;

use App\Services\PaymentService;
use Illuminate\Support\ServiceProvider;
use Illuminate\Contracts\Foundation\Application;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('payment.service', function (Application $app) {
            $paymentService = new PaymentService();

            $paymentService->setTaxService($app['tax.service']);

            // Will set a new TaxService instance for the Payment Service
            $app->refresh('tax.service', $paymentService, 'setTaxService');

            return $paymentService;
        });
    }
}
```

## Laravel Tip 💡: Bind Typed Variadics ([⬆️](#container-tips-cd-))

Did you know that you can bind typed variadics to the container? Laravel ships with 3 methods to allow you to do so: "when()", "needs()", and "give()". You can keep using DI without worries! 🚀

```php
<?php

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
    protected array $filters;

    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}

// In the service provider
$this->app->when(Firewall::class)
    ->needs(Filter::class)
    ->give(function (Application $app) {
        return [
            $app->make(NullFilter::class),
            $app->make(ProfanityFilter::class),
            $app->make(TooLongFilter::class),
        ];
    });
```

## Laravel Tip 💡: Check Your Application Enviroment ([⬆️](#container-tips-cd-))

We often need to check the application environment. While you can use the environment method to do so, Laravel ships with elegant methods "isProduction" and "isLocal" to do exactly that 🚀

```php
<?php

// Could be better 😫
app()->environment() === 'production'

// Okay 😐
app()->environment('production')

// Better 😎
app()->isProduction()

// Also applies for 'local' environment with "isLocal()"
```

## Laravel Tip 💡: The New "optimizes" Method ([⬆️](#container-tips-cd-))

Laravel v11.27.1 introduced a new service provider method called "optimizes". You can now define commands to run with the "optimize" and "optimize:clear" commands, such as "filament:optimize" or other custom commands 🚀

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $this->optimizes(
            optimize: 'filament:optimize',
            // Defaults to the service provider name without "ServiceProvider" suffix
            key: 'filament' 
        );

        // ...
    }
}
```

## Laravel Tip 💡: Deferred Providers ([⬆️](#container-tips-cd-))

If you have a service provider that only registers some bindings, you can mark it as deferred by implementing the "DeferrableProvider" interface. This way, it will load only when one of its bindings is needed 🚀

```php
<?php
 
namespace App\Providers;
 
use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

// This service provider will only load when the Connection is needed,
// improving your apps performance.
class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }
}
```
