# Production Oriented Laravel Project Setup Quickstart

May 24, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Production Oriented Laravel Project Setup Quickstart](https://aregsar.com/blog/2020/production-oriented-laravel-project-setup-quickstart)

In this article I am going to show my setup procedure to setup a new Laravel project with the following out of the box features:

- User  Authentication
- Tailwind UI
- Docker services for local development (mysql, redis, mailhog, elasticsearch)
- Test MySQL database docker service for running feature tests
- Configuration for Application, Session, Cache and Queue to use Redis locally and in production
- script to start the docker services
- user verification and password reset sent as background jobs

## Seting up a new Laravel project

First create a new project as instructed here:

Make sure php-redis extension is installed in your local environment.

Installation instructions for php-redis extension can be found at https://github.com/phpredis/phpredis/blob/develop/INSTALL.markdown and https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster

```bash
pecl install --force redis
pecl list
php -m
```

```bash
composer create-project --prefer-dist laravel/laravel myapp
cd myapp && php artisan --version
git init

composer require laravel-frontend-presets/tailwindcss --dev
php artisan ui tailwindcss --auth
npm install && npm run dev

git add -A
git commit -m "first commit"
git remote add origin git@github.com:aregsar/myapp.git
git push -u origin master
```

Also check the extension exists at /usr/local/lib/php/pecl/20190902/redis.so and check the top of the php.ini file at /usr/local/etc/php/7.4/php.ini includes the line extension="redis.so".
If it does not exists, make sure to add it manually so PECL or PHP detect the extension.

## Create the data directory

Create a new data directory in the Laravel project root directory:

```bash
echo '/data' >> .gitignore
mkdir data
```

## Create the docker-compose file

Create a new docker-compose.yml file in the Laravel project root directory:

```bash
touch docker-compose.yml
```

Next add the following content to the docker-compose.yml file

```yaml
version: "3.1"
services:

    testmysql:
      image: mysql:8.0
      container_name: app-test-mysql
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        - "8011:3306"

    mysql:
      image: mysql:8.0
      container_name: app-mysql
      volumes:
        - ./data/mysql:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        - "8001:3306"

    redis:
      image: redis:alpine
      container_name: app-redis
      command: redis-server --appendonly yes --requirepass "${REDIS_PASSWORD}"
      volumes:
      - ./data/redis:/data
      ports:
        - "8002:6379"

    mailhog:
      image: mailhog/mailhog:latest
      container_name: app-mailhog
      ports:
        - "8003:1025"
        - "8025:8025"

    elasticsearch:
      image: elasticsearch:6.5.4
      container_name: app-elasticsearch
    ports:
        - "8004:8099"
```

## Replace the .env file content

```ini
APP_NAME=myapp
APP_ENV=local
APP_KEY=base64:Ht9uVlWv7mf+mZfRRjtz2RqkoZ/Dfo2dVK3pcKUBDT8=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

BROADCAST_DRIVER=log

CACHE_DRIVER=redis

SESSION_DRIVER=redis
SESSION_LIFETIME=120

QUEUE_CONNECTION=job
QUEUE_PREFIX_VERSION=V1:

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
TEST_DB_PORT=8011

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=radar
REDIS_PORT=8002

MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"
```

## Replace config/database.php

```php
<?php

use Illuminate\Support\Str;

return [

    'default' => env('DB_CONNECTION', 'mysql'),

    'connections' => [

        'sqlite' => [
            'driver' => 'sqlite',
            'url' => env('DATABASE_URL'),
            'database' => env('DB_DATABASE', database_path('database.sqlite')),
            'prefix' => '',
            'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
        ],

        'mysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '8001'),
            'database' => env('DB_DATABASE', 'myapp'),
            'username' => env('DB_USERNAME', 'myapp'),
            'password' => env('DB_PASSWORD', 'myapp'),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],

        'testmysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('TEST_DB_PORT', '8011'),
            'database' => env('DB_DATABASE', 'myapp'),
            'username' => env('DB_USERNAME', 'myapp'),
            'password' => env('DB_PASSWORD', 'myapp'),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],

        'pgsql' => [
            'driver' => 'pgsql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '5432'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8',
            'prefix' => '',
            'prefix_indexes' => true,
            'schema' => 'public',
            'sslmode' => 'prefer',
        ],

        'sqlsrv' => [
            'driver' => 'sqlsrv',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', 'localhost'),
            'port' => env('DB_PORT', '1433'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8',
            'prefix' => '',
            'prefix_indexes' => true,
        ],
    ],

    'migrations' => 'migrations',

    //the redis driver
    'redis' => [
        //the redis client (requires installing the phpredis.so extension using pecl)
        'client' => 'phpredis',

        'options' => [
            //this setting is only effective when using a managed redis cluster. No impact if redis cluster is not used.
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],
        //connection used by the redis facade
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'd:',
        ],
        //connection used by the cache facade when redis cache is configured in config/cache.php
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],
        //connection used by the session when redis cache is configured in config/session.php
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 's:',
        ],
        //connection used by the queue when redis cache is configured in config/queue.php
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:'.env('QUEUE_PREFIX_VERSION', ''),
        ],
    ],
];
```

## Replace config/cache.php

```php
<?php

use Illuminate\Support\Str;

return [

    'default' => env('CACHE_DRIVER', 'redis'),

    'stores' => [

        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
        ],

        'apc' => [
            'driver' => 'apc',
        ],

        'array' => [
            'driver' => 'array',
            'serialize' => false,
        ],

        'database' => [
            'driver' => 'database',
            'table' => 'cache',
            'connection' => null,
        ],

        'file' => [
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],

        'memcached' => [
            'driver' => 'memcached',
            'persistent_id' => env('MEMCACHED_PERSISTENT_ID'),
            'sasl' => [
                env('MEMCACHED_USERNAME'),
                env('MEMCACHED_PASSWORD'),
            ],
            'options' => [
                // Memcached::OPT_CONNECT_TIMEOUT => 2000,
            ],
            'servers' => [
                [
                    'host' => env('MEMCACHED_HOST', '127.0.0.1'),
                    'port' => env('MEMCACHED_PORT', 11211),
                    'weight' => 100,
                ],
            ],
        ],

        'dynamodb' => [
            'driver' => 'dynamodb',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
            'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
            'endpoint' => env('DYNAMODB_ENDPOINT'),
        ],
    ],

    'prefix' => '',
];
```

## Replace config/session.php

```php
<?php

use Illuminate\Support\Str;

return [

    'driver' => env('SESSION_DRIVER', 'redis'),

    'connection' => 'session',

    'lifetime' => env('SESSION_LIFETIME', 120),

    'expire_on_close' => false,

    'encrypt' => false,

    'files' => storage_path('framework/sessions'),

    'table' => 'sessions',

    'store' => env('SESSION_STORE', null),

    'lottery' => [2, 100],

    'cookie' => env(
        'SESSION_COOKIE',
        Str::slug(env('APP_NAME', 'laravel'), '_').'_session'
    ),

    'path' => '/',

    'domain' => env('SESSION_DOMAIN', null),

    'secure' => env('SESSION_SECURE_COOKIE'),

    'http_only' => true,

    'same_site' => 'lax',
];
```

## Replace config/queue.php

```php
<?php

return [

    'default' => env('QUEUE_CONNECTION', 'job'),

    'connections' => [
        //this is the 'job' queue connection
        'job' => [
            //uses the redis driver from config/database.php
            'driver' => 'redis',
            //uses the 'queue' connection from the redis driver in config/database.php
            'connection' => 'queue',
            //this is the redis queue key default prefix that is applied when using this 'job' connection. It can be overriden by explicitly passing the queue name.
            'queue' => '{job}',
            'retry_after' => 90,
            'block_for' => null,
        ],
        //this is the 'app' queue connection
        'app' => [
            //uses the redis driver from config/database.php
            'driver' => 'redis',
            //uses the 'queue' connection from the redis driver in config/database.php
            'connection' => 'queue',
            //this is the redis queue key default prefix that is applied when using this 'app' connection.It can be overriden by explicitly passing the queue name.
            'queue' => '{app}',
            'retry_after' => 90,
            'block_for' => null,
        ],

        'sync' => [
            'driver' => 'sync',
        ],

        'database' => [
            'driver' => 'database',
            'table' => 'jobs',
            'queue' => 'default',
            'retry_after' => 90,
        ],

        'beanstalkd' => [
            'driver' => 'beanstalkd',
            'host' => 'localhost',
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => 0,
        ],

        'sqs' => [
            'driver' => 'sqs',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'prefix' => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
            'queue' => env('SQS_QUEUE', 'your-queue-name'),
            'suffix' => env('SQS_SUFFIX'),
            'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        ],
    ],

    'failed' => [
        'driver' => env('QUEUE_FAILED_DRIVER', 'database'),
        'database' => env('DB_CONNECTION', 'mysql'),
        'table' => 'failed_jobs',
    ],
];
```

## Startup the containers


In the project root directory, type:

```bash
docker-compose up -d
docker ps -a
php artisan tinker
```

## Check database connections

Start Laravel tinker to check the MySQL and Redis connections:

```bash
php artisan tinker
```

Check the default MySQL connection:

```php
DB::connection()->getPdo();
```

Check the resdis connection:

```php
Illuminate\Support\Facades\Redis::connection()->ping();
```

Check the MySQL test database connection:

```php
DB::connection("testmysql")->getPdo();
```

## configure email

Update the env vars for connecting to MailHog running in docker:

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=8003
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"
```

The `config/mail.php` file is where these env vars are used:

```php
'mailers' => [
      'smtp' => [
          'transport' => 'smtp',
          'host' => env('MAIL_HOST', 'localhost'),
          'port' => env('MAIL_PORT', 8003),
          'encryption' => env('MAIL_ENCRYPTION', 'tls'),
          'username' => env('MAIL_USERNAME'),
          'password' => env('MAIL_PASSWORD'),
          'timeout' => null,
      ],
],
  'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'admin@myapp.com'),
        'name' => env('MAIL_FROM_NAME', 'Admin'),
    ],
```

Connect to the MailHog admin dashboard using your browser:

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome localhost:8025
```

## Queue Verification Email setup

In `routes\web.php` file change:

```php
Auth::routes();
```

To:

```php
Auth::routes(['verify' => true]);
```

Run the artisan command to create a new notification to override the default Email Verification notification:

```bash
artisan make:notification Auth/QueuedVerifyEmail
```

Open the new notification class file at `App/Notifications/Auth/QueuedVerifyEmail.php` and replace its content with the following:

```php
namespace App\Notifications\Auth;

use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Bus\Queueable;

class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;

    public function __construct()
    {
        //Uncomment to set custom queue for this notification
        //$this->queue = 'mycustomequeue';

        //Uncomment to set custom queue connection for this notification
        //$this->connection = 'mycustomequeueconnection';
    }
}
```

Finally Replace `App\User.php` content with the following:

```php
<?php

namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

//class User extends Authenticatable
class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

     /**
     * Overrideen sendEmailVerificationNotification implementation
     *
     * @var array
     */
    public function sendEmailVerificationNotification()
    {
        $this->notify(new App\Notifications\Auth\QueuedVerifyEmail);
    }

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
}
```

Here we have added the `implements MustVerifyEmail` to the `class User extends Authenticatable` class definition and we have also added the `sendEmailVerificationNotification` method  that overrides the default method that the User class inherits to the User class body.



## Run database migrations

```bash
php artisan migrate
```

To run  migrations on the command line against the test database before running tests:

```bash
php artisan migrate --database=testmysql
```

To run migrations using the `RefreshDatabase` trait before running tests:
override the .env file `DB_CONNECTION` setting in phpunit.xml

```xml
<name="DB_CONNECTION" value="testmysql">
```

## Serve the application using Valet

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome myapp.test
```

## Serve the application using Artisan

```bash
php artisan serve
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome localhost:8000
```

## Run php unit tests

```bash
phpunit
```

## Run parallel tests with Paratest package