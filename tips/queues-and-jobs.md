# Queues & Jobs Tips ([cd ..](../README.md))

- [Dispatch After Response](#laravel-tip--dispatch-after-response-️)
- [Delete a Job When Models Are Missing](#laravel-tip--delete-a-job-when-models-are-missing-️)
- [Clean Up After Failed Jobs](#laravel-tip--clean-up-after-failed-jobs-️)
- [Skip Relationships in Queues](#laravel-tip--skip-relationships-in-queues-️)
- [Bulk Dispatch](#laravel-tip--bulk-dispatch-️)
- [Skip Jobs When Batch is Cancelled](#laravel-tip--skip-jobs-when-batch-is-cancelled-️)
- [Skip Jobs](#laravel-tip--skip-jobs-️)
- [Keeping Jobs Unique Until Processing](#laravel-tip--keeping-jobs-unique-until-processing-️)
- [Rate Limit Jobs](#laravel-tip--rate-limit-jobs-️)

## Laravel Tip 💡: Dispatch After Response ([⬆️](#queues--jobs-tips-cd-))

At times, certain tasks, like sending emails, don't necessarily need to be queued and processed by a worker. In such cases, you can make use of "dispatchAfterResponse()." True to its name, this method dispatches the job right after the server responds to the user. A quick response for the client without burdening the worker with minor tasks 🚀

```php
<?php

use App\Jobs\SendNotification;

SendNotification::dispatchAfterResponse();
```

## Laravel Tip 💡: Delete a Job When Models Are Missing ([⬆️](#queues--jobs-tips-cd-))

Eloquent models injected into jobs are auto-serialized for the queue and may trigger a `ModelNotFoundException` if deleted while waiting for a worker; you can quietly discard such jobs by setting `deleteWhenMissingModels` to true 🚀

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue
{
    /**
     * Delete the job if its models no longer exist.
     *
     * @var bool
     */
    public $deleteWhenMissingModels = true;
}
```

## Laravel Tip 💡: Clean Up After Failed Jobs ([⬆️](#queues--jobs-tips-cd-))

When jobs fail, you may want to send notifications or perform some cleanups. Luckily, Laravel allows you to define a "failed" method to do exactly that 🚀

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Throwable;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    // ...

    public function failed(?Throwable $exception): void
    {
        // Handle job failure, such as sending notifications
    }
}
```

## Laravel Tip 💡: Skip Relationships in Queues ([⬆️](#queues--jobs-tips-cd-))

When passing a model to a job, consider using the "WithoutRelations" attribute to skip serializing the relationships if you don't need them. This will keep the payload minimal and memory efficient 🚀

```php
<?php

use Illuminate\Queue\Attributes\WithoutRelations;

public function __construct(
    #[WithoutRelations]
    public Podcast $podcast
) {}
```

## Laravel Tip 💡: Bulk Dispatch ([⬆️](#queues--jobs-tips-cd-))

While Laravel offers batches to dispatch jobs, sometimes you just want to fire and forget. In that case, you can bulk dispatch instead of doing it one by one 🚀

```php
<?php

// If you are using the database driver, this will result in N queries
$users->each(fn(User $user) => GenerateInvoice::dispatch($user));

// This will result in a single query
$jobs = $users->map(fn(User $user) => new GenerateInvoice($user))->toArray();
Queue::bulk($jobs);
```

## Laravel Tip 💡: Skip Jobs When Batch is Cancelled ([⬆️](#queues--jobs-tips-cd-))

When working with batched jobs, it's best to check if the batch is canceled before running the job, and you don't have to do it manually because the "SkipIfBatchCancelled" middleware does exactly this 🚀

```php
<?php
use App\Jobs\ImportContacts;
use Illuminate\Support\Collection;

// A batched job
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }
}

public function middleware(): array
{
    return [new SkipIfBatchCancelled];
}
```

## Laravel Tip 💡: Skip Jobs ([⬆️](#queues--jobs-tips-cd-))

Ever needed to skip job execution? While you can do it manually, Laravel ships with a "Skip" middleware to do exactly that 🚀

```php
<?php

use Illuminate\Queue\Middleware\Skip;

public function middleware(): array
{
    return [
        Skip::when($someCondition),
        // You can also use `unless` and pass a closure
        Skip::unless(fn () => $this->shouldNotSkip()),
    ];
}
```

## Laravel Tip 💡: Keeping Jobs Unique Until Processing ([⬆️](#queues--jobs-tips-cd-))

Sometimes, you may have jobs where only one job can be queued at a time, but once it starts processing, you can queue more jobs. Laravel ships with the "ShouldBeUniqueUntilProcessing" contract to do exactly this 🚀

```php
<?php
 
use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;
 
class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

## Laravel Tip 💡: Rate Limit Jobs ([⬆️](#queues--jobs-tips-cd-))

Have you ever needed to rate-limit a job? Whether to avoid overwhelming an API or to limit free plan users from running too many jobs, Laravel allows you to define rate limiters and use them out of the box 🚀

```php
<?php

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Queue\Middleware\RateLimited;

// In one of your service providers
RateLimiter::for('reports', function (object $job) {
    return $job->user->vipCustomer()
        ? Limit::none()
        : Limit::perHour(5)->by($job->user->id);
});


// In your job class
public function middleware(): array
{
    return [new RateLimited('reports')];
}
```
