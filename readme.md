# JWT Auth and Roles

[![Latest Version on Packagist][ico-version]][link-packagist]
[![Total Downloads][ico-downloads]][link-downloads]
[![MIT Licensed](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](license.md) [![Join the chat at https://gitter.im/werk365/Laravel-JWT-Auth-Roles](https://badges.gitter.im/werk365/Laravel-JWT-Auth-Roles.svg)](https://gitter.im/werk365/Laravel-JWT-Auth-Roles?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

![StyleCI][ico-styleci]
[![Scrutinizer Quality][ico-scrutinizer]][link-scrutinizer]
![Tests](https://github.com/365Werk/Laravel-JWT-Auth-Roles/workflows/Run%20Tests/badge.svg)


Made to use JWTs from an external identity provider in Laravel. Tested with Fusionauth, but should be quite general purpose.

With this package you can validate the incoming JWT, and create an authenticated user that has to roles specified in the JWT for further (route based) authentication using a role middleware that is included.

.

Take a look at [contributing.md](contributing.md) to see a to do list.

## Installation

Via Composer

``` bash
$ composer require werk365/jwtauthroles
```

Publish config and migration

```bash
$ php artisan vendor:publish --provider="Werk365\JwtAuthRoles\JwtAuthRolesServiceProvider"
```

Migrations are only needed if you want to either cache the JWKs or store the user, this can be configured in the config. It's possible to use this package without storing anything related to it in the database at all.

Run migration
```bash
$ php artisan migrate
```

## Usage

In your AuthServiceProvider modify boot()
```php
use Illuminate\Support\Facades\Auth;
use Werk365\JwtAuthRoles\JwtAuthRoles;

public function boot()
{
    $this->registerPolicies();

    Auth::viaRequest('jwt', function ($request) {
        return JwtAuthRoles::authUser($request);
    });
}
```

Then either change one of your guards in config/auth.php to use the jwt driver and jwt_users provider, or add a new guard
```php
use Werk365\JwtAuthRoles\Models\JwtUser;
'guards' => [
    // ...
    'jwt' => [
        'driver' => 'jwt',
        'provider' => 'jwt_users',
        'hash' => false,
    ],
],

// ...

'providers' => [
    // ...
    'jwt_users' => [
        'driver' => 'eloquent',
        'model' => JwtUser::class,
    ],
],
```

Now you can use the JWT guard in your routes, for example on a group:
```php
Route::group(['middleware' => ['auth:jwt']], function () {
    // Routes can go here
});
```

You can also use the RolesMiddelware to do role-based authentication on a route like this:
```php
    // single role
    Route::get('/exammple', function(){
        return "example";
    })->middleware('role:example');

    // multiple roles
    Route::get('/exammples', function(){
        return "examples";
    })->middleware('role:example|second|third|etc');
```

To make the authenticated user actually useful, the JwtUser model extends the User model. This means that you can define any relations in the User model, and then use them for the authenticated user.

For example, add the following relationship in the default User model:
```php
    public function documents()
    {
        return $this->hasMany('App\Models\Document', 'user', 'uuid');
    }
```
This assumes you have a Documents model where the uuid provided by your identity provider is stored in a 'user' column, this can be anything you want of course, but the local key should always be uuid.

This can then be used as follows to retrieve all documents belonging to this user:

```php
return Auth::user()->documents;
```

Finally, configure the config to your needs. The default published config will validate the JWT, but not use the database. It looks like this:
```php
<?php

return [
    // If enabled, stores every user in the database
    'useDB' => env('FA_USE_DB', false),

    // Only if useDB = true
    // Column name in the users table where uuid should be stored.'
    'userId' => env('FA_USR_ID', 'uuid'),
    // Only if useDB = true
    'autoCreateUser' => env('FA_CREATE_USR', false),

    'alg' => env('FA_ALG', 'RS256'),

    // Allows you to skip validation, this is potentially dangerous,
    // only use for testing or if the jwt has been validated by something like an api gateway
    'validateJwt' => env('FA_VALIDATE', true),

    // Only if validateJwt = true
    'cache' => [
        'enabled' => env('FA_CACHE_ENABLED', false),
        'type' => env('FA_CACHE_TYPE', 'database'),
    ],

    // Only if validateJwt = true
    'jwkUri' => env('JWKS_URL', 'http://localhost:9011/.well-known/jwks.json'),
    // Only if validateJwt = true
    'pemUri' => env('PEM_URL', 'http://localhost:9011/api/jwt/public-key'),

    // Only if validateJwt = true
    // Configure to use PEM endpoint (default) or JWK
    'useJwk' => env('USE_JWK', false),

];
```

## Laravel version
Currently this package supports Laravel 8. Since we use the default User model, it expects it to be in the app\Models\User namespace. To make this package work with previous versions of Laravel, you'll only have to make a model in this namespace, besides that the package should work with any recent version.

## Change log

Please see the [changelog](changelog.md) for more information on what has changed recently.

## Testing

Testing is not yet implemented

## Contributing

Please see [contributing.md](contributing.md) for details and a todolist.

## Security

If you discover any security related issues, please email <hergen.dillema@gmail.com> instead of using the issue tracker.

## Credits

- [Hergen Dillema][link-author]
- [All Contributors][link-contributors]

## License

license. Please see the [license file](license.md) for more information.

[ico-version]: https://img.shields.io/packagist/v/werk365/jwtauthroles.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/werk365/jwtauthroles.svg?style=flat-square
[ico-travis]: https://img.shields.io/travis/werk365/jwtauthroles/master.svg?style=flat-square
[ico-styleci]: https://github.styleci.io/repos/278075608/shield
[ico-scrutinizer]: https://scrutinizer-ci.com/g/365Werk/Laravel-JWT-Auth-Roles/badges/quality-score.png

[link-packagist]: https://packagist.org/packages/werk365/jwtauthroles
[link-downloads]: https://packagist.org/packages/werk365/jwtauthroles
[link-scrutinizer]: https://scrutinizer-ci.com/g/365Werk/Laravel-JWT-Auth-Roles/
[link-author]: https://github.com/HergenD
[link-contributors]: ../../contributors
