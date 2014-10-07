# AltoRouter [![Build Status](https://api.travis-ci.org/dannyvankooten/AltoRouter.png)](http://travis-ci.org/dannyvankooten/AltoRouter)
AltoRouter is a small but powerful routing class for PHP 5.3+, heavily inspired by [klein.php](https://github.com/chriso/klein.php/).

* Dynamic routing with named parameters
* Reversed routing
* Flexible regular expression routing (inspired by [Sinatra](http://www.sinatrarb.com/))
* Custom regexes

## Getting started

1. PHP 5.3.x is required
2. Install AltoRouter using Composer or manually
2. Setup URL rewriting so that all requests are handled by **index.php**
3. Create an instance of AltoRouter, map your routes and match a request.
4. Have a look at the basic example in the `examples` directory for a better understanding on how to use AltoRouter.

## Routing

With AltoRouter you define new routes by using the `map` method.

`map` accepts 4 parameters;

**Request methods**  
One of 5 HTTP Methods, or a pipe-separated list of multiple HTTP Methods (GET|POST|PATCH|PUT|DELETE)

**Route pattern**  
Route pattern to match with. You can use multiple pre-set regex filters, like `[i:id]`. A custom regex must start with an `@`.

**Target**  
The target of where this route should point to. Can be anything.

**Name**  
Optional name of this route. Supply if you want to reverse route this url in your application.

### Example

```php
$router = new AltoRouter();
// (optional) set basepath to the subdirectory relative to the application root
// or, a virtual path where you want to integrate AltoRouter.
$router->setBasePath('/AltoRouter');

$router->map('GET|POST','/', 'home#index', 'home');
$router->map('GET','/users', array('c' => 'UserController', 'a' => 'ListAction'));
$router->map('GET','/users/[i:id]', 'users#show', 'users_show');
$router->map('POST','/users/[i:id]/[delete|update:action]', 'usersController#doAction', 'users_do');

// reversed routing
$router->generate('users_show', array('id' => 5)); # => /users/5
```

For quickly adding multiple routes, you can use the `addRoutes` method. This method accepts an array or any kind of traversable.

```php
$router->addRoutes(array(
  array('PATCH','/users/[i:id]', 'users#update', 'update_user'),
  array('DELETE','/users/[i:id]', 'users#delete', 'delete_user')
));
```

**You can use the following limits on your named parameters. AltoRouter will create the correct regexes for you.**

```php
*                    // Match all request URIs
[i]                  // Match an integer
[i:id]               // Match an integer as 'id'
[a:action]           // Match alphanumeric characters as 'action'
[h:key]              // Match hexadecimal characters as 'key'
[:action]            // Match anything up to the next / or end of the URI as 'action'
[create|edit:action] // Match either 'create' or 'edit' as 'action'
[*]                  // Catch all (lazy, stops at the next trailing slash)
[*:trailing]         // Catch all as 'trailing' (lazy)
[**:trailing]        // Catch all (possessive - will match the rest of the URI)
.[:format]?          // Match an optional parameter 'format' - a / or . before the block is also optional
```

**Some more complicated examples**

```php
@/(?[A-Za-z]{2}_[A-Za-z]{2})$ // custom regex, matches language codes like "en_us" etc.
/posts/[*:title][i:id]        // Matches "/posts/this-is-a-title-123"
/output.[xml|json:format]?    // Matches "/output", "output.xml", "output.json"
/[:controller]?/[:action]?    // Matches the typical /controller/action format
```

**The character before the colon (the 'match type') is a shortcut for one of the following regular expressions**

```php
'i'  => '[0-9]++'
'a'  => '[0-9A-Za-z]++'
'h'  => '[0-9A-Fa-f]++'
'*'  => '.+?'
'**' => '.++'
''   => '[^/\.]++'
```

**New match types can be added using the `addMatchTypes()` method**

```php
$router->addMatchTypes(array('cId' => '[a-zA-Z]{2}[0-9](?:_[0-9]++)?'));
```

## Matching

Route lookup is done by calling `match`.

```php
// perform a match against the current request url
$result = $router->match();

// perform a match against a given url
$result = $router->match('/path/to/somewhere');
```

### Return value

The return value will be an associative array if a match is found and `false` otherwise.

The array consists of 3 keys; `target`, `params` and `name`.

Where `target` and `name` contain the values passed to `map` and `params` is an associative array of params extracted from the url.

### Example

```php
$router = new AltoRouter();
$router->map('GET', '/user/[i:id]', 'user_controller#show_profile', 'user_profile');
$result = $router->match('/user/123');

$result == array(
  'target' => 'user_controller#show_profile',
  'params' => array(
    'id' => 123
  ),
  'name' => 'user_profile'
);
```

## Contributors
- [Danny van Kooten](https://github.com/dannyvankooten)
- [Koen Punt](https://github.com/koenpunt)
- [John Long](https://github.com/adduc)
- [Niahoo Osef](https://github.com/niahoo)

## License

(MIT License)

Copyright (c) 2012-2013 Danny van Kooten <hi@dannyvankooten.com>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
