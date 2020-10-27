# Laravel Tips

October 27, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Laravel Tips](https://aregsar.com/blog/2020/laravel-tips)

In this post I list a bunch of ad hoc Laravel tips for future reference.

## Useful packages

`beyondcode/laravel-query-detector`

`barryvdh/laravel-debugbar`

## Run the Laravel Scheduler with single crontab

```php
#need this command added to crontab file on production server for laravel scheduler to run
* * * * * cd /PATH-TO-LARAVEL-APP && php artisan schedule:run >> /dev/null 2>&1
```

## Laravel blade php variables

```php
//declare a variable
@php
$bv = $data['bv']
@endphp

//declare a variable in a single line
@php ($bv = $data['bv'])

//declare a variable in a single line
@php $myVar = 'My Value';
```

## Remove logic from blade views

Sometimes we have conditional code based on some model state, like so:

```php
@if($post->ownedBy(auth()->user()))
```

Or like so:

```php
@if(auth()->user()->ownsPost($post))
```

This logic is hard to test.

Instead do this logic in a controller action, in the model or a helper class.

Set the result to a variable and pass that variable to the view.

Then the conditional logic will be simplified as:

```php
@if($post_owned_by_current_user)
```

Which means when we write feature tests we can just check the value of this variable returned from the action.

> Note: if we have a collection of posts and have to do this check on each post, then we loop through the posts in the action and calculate the values for all the posts and return them as an additional variable for each post.

## Use helper class for model and model collection operations

To keep your models focused on the data and not mix in a bunch of operations that may have unrelated functionality we can have a bunch of focused operation classes that take models or model collections and operate on them.

```php
//get back and array of closures one for each user
$jobs = Users::onBasePlan()->map(fn($user){
    return fn() use ($user){
        $user->upgrade();
    };
});

//run a batch of jobs each job will upgrade a specific user
Bus::batch($jobs)->dispatch();
```

Using a `PlanFilter` helper for the `onBasePlan` method instead:

```php
//get back and array of closures one for each user
$jobs = PlanFilter::UsersOnBasePlan(Users)->map(fn($user){
    return fn() use ($user){
        $user->upgrade();
    };
});

//run a batch of jobs each job will upgrade a specific user
Bus::batch($jobs)->dispatch();
```

## In memory vs database collections

difference between Illuminate\Support\Collection and Illuminate\Database\Eloquent\Collection

## Eloquent relations vs data access methods

difference between model->relproperty->model vs model->relmethod()->model in eloquent relations when evaluated

## Request properties and PHP magic methods

accessing model->property in request and form request objects using magic get methods