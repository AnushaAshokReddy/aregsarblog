# Laravel App Sending Email Using Background Job

[Laravel App Sending Email Using Background Job](https://aregsar.com/blog/2020/laravel-app-sending-email-using-background-job)

In this article I will show you how we can created a background job that sends an email using the Laravel Mailable class and queue that job to send the email in the background using a Redis backed queue.

> I have covered configuring Laravel queue to use redis here xxxx. So I will not go over that part here.


## Creating the Mailable

```ini
php artisan make:mail WelcomeEmail
```

```php
<?php
  
namespace App\Mail;
  
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;
  
class WelcomeEmail extends Mailable
{
    use Queueable, SerializesModels;
  
    public function __construct()
    {
    }
  
    public function build()
    {
        return $this->from('admin@app.com', 'Admin')
                    ->subject('Welcome')
                    ->view('mails.welcomeemail')
                    ->text('mails.welcomeemail-text');
    }
}
```

## Creating the Email Job

```ini
php artisan make:job WelcomeEmailJob
```


```php
<?php
  
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use App\Mail\WelcomeEmail;
use Mail;

class WelcomeEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
  
    protected $to;
  
    public function __construct($email)
    {
        $this->to = $email;
    }

    /**
     * Execute the job.
     */
    public function handle()
    {
        Mail::to($this->to)->send(new WelcomeEmail);
    }
}
```


## Dispatching the Email Job

Add the following route

```php
Route::get('welcome', 'WelcomeEmailController@send');
```

Add the following controller

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Controllers\Controller;

use App\Jobs\WelcomeEmailJob;

class WelcomeEmailController extends Controller
{
    public function send(Request $request)
    {
        WelcomeEmailJob::dispatch('recipient@example.com');

        //Alternative way of dispaching Job
        //dispatch(new WelcomeEmailJob('recipient@example.com'));

        //override the default connection
        //override the default queue of the connection
        //you can override both or one or the other
        //WelcomeEmailJob::dispatch('recipient@example.com')->onConnection('sqs')->onQueue('processing');
    }
}
```

## Processing the queued emails

```ini
php artisan queue:work --tries=3 --timeout=30

```

## Further customizations

For further customizations see `https://blog.mailtrap.io/laravel-mail-queue/`