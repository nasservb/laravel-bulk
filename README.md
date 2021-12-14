# Laravel Bulk Operation Tool
[![Latest Stable Version](https://poser.pugx.org/nasservb/laravel-bulk/v)](//packagist.org/packages/nasservb/laravel-bulk) [![Total Downloads](https://poser.pugx.org/nasservb/laravel-bulk/downloads)](//packagist.org/packages/nasservb/laravel-bulk) [![License](https://poser.pugx.org/nasservb/laravel-bulk/license)](//packagist.org/packages/nasservb/laravel-bulk) 

This package provides Bulk operation for Laravel projects.

## Contents

- [Installation](#installation)
- [Usage](#usage)
- [Credits](#credits)
- [License](#license)

## Installation

You can install the package via composer:

```bash
composer require nasservb/laravel-bulk
```

Laravel will automatically register the package service provider.

#### Install redis client

For use this library we recomend using the latest version of Redis at this time `(^6)`

```bash
composer require predis/predis
```

### Setting up Redis configuration

After you've published the Laravel Bulk package configuration, you need to set your driver to `Redis` and add its configuration:

```bash
// .env
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379

```


## Usage

### What's the Problem and How to solve it?

when you need to save data to the DB, and this data is obtained over time such as page visit count or the number of like, it is not a good idea to insert data into the database for each request.

We can use Redis to cache this data, and when the number of items reaches a significant amount, read the data from Redis and save it to DB via a bulk insert query.

This PR helps us to do this simply in a few lines of code for saving visits:
```php 
	app('bulk')
	    ->name('page-visit')
	    ->add(request()->ip())
	    ->then(function($visits)use($page){
		$page->visits()->attach($visits);
	    });
```
or for saving likes:
```php 
	app('bulk')
	    ->name('like-count')
            ->add(auth()->id())
            ->then(function ($likes) use ($post) {
                $post->likes()->sync($likes);
            });
```

### How does this work internally?
- This code uses the Redis RPUSH command to add data to the cache,
- Check cache size with Redis LLEN command,
- And when the cache is full, release them with the Redis LRANGE command and delete the cache items with the Redis DEL command.

### What are the benefits?
If you have a direct insert in each request and replace it with the Redis::bulk function, your performance will be 10 times better in the default configuration. you can configure your cache size with:
```php 
	app('bulk')
	    ->name('name')
            ->count(cache_size); 
            ...
```

for saving likes:
```php 
	app('bulk')
	    ->name('like-count')
            ->add(auth()->id())
            ->count(100)
            ->then(function ($likes) use ($post) {
                $post->likes()->sync($likes);
            });
```
### How to extract cache data?
And improve the performance again, but keep in mind that when the cache is large, saving your data will be delayed until the cache is full and remains in memory.

When the execution of your code is going to finish, you need to release data from the cache and save it to DB.

To do this, you can use the forceRelease function as follows:
```php 
	app('bulk')
	    ->name('like-count')
            ->forceRelease()
            ->then(function ($likes) use ($post) {
                $post->likes()->sync($likes);
            });
```
You can add this code in Application::shutdown() function or any other place that runs at the end.

## Limitations

it's depend on cach size and redis capablility.

## Credits

- [Nasser Niazy](https://github.com/nasservb)
- [All Contributors](../../contributors)

## License

The MIT License (MIT).

