# Cool API (by Robert Grubb)

A new PHP API framework that is simple and fast. I forked this repo for my private project.

## Installation

Make sure you have rewrite mod enabled, and you place the following in `.httaccess` where your public folder is located. **NOTE:** CoolApi will throw an exception if it's not found.

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```

**NOTE: The API will attempt to do this soon for you if the directory is writable**

`composer require idammi/coolapi`

## Configuration Explained

All values below are the defaults set by the API.

```
$config = [

  /**
   * The baseUri is necessary if you are in a sub-directory on
   * hosting. An example is: http://localhost/foo/bar/api
   *
   * If you specify /foo/bar/api in baseUri, it will be removed
   * from the URI when matching routes.
   */
  'baseUri' => '/',

  /**
   * Log configuration:
   *
   * Logger requires a path that is writable, and the API will
   * throw an exception if it is not. Below is the configuration
   * for the logger.
   */
  'logging' => [
    'enabled' => true,
    'path'    => \CoolApi\Core\Utilities::root() . '/logs/',
    'file'    => 'api.log'
  ],

  /**
   * Configuration for cors
   */
  'cors'    => [
    'enabled'   => true,
    'whitelist' => '*',
    'blacklist' => false
  ],

  /**
   * Configuration for api keys
   *
   * With CoolApi, you can require api keys for access to your
   * API. Look below for the configuration.
   *
   * CoolApi looks for the api key in 3 places:
   * - Authorization: Bearer <api_key_here>
   * - $_GET['key']
   * - $_POST['key']
   *
   * Will accept a valid key from any of the above locations.
   */
  'apiKeys'    => [
    'enabled'  => false,
    'keyField' => 'key', // What it looks for in $_GET or $_POST
    'keys'     => [] // Array of key strings
  ],

  /**
   * Configuration for Rate Limiting
   *
   * CoolApi comes with an out-of-the-box solution for rate limiting.
   * In order for this to work properly, you must:
   * 1. Provide a storage path (Look below for storage config) for FilerDB.
   * 2. Enable it, and set a limit per window.
   */
  'rateLimit' => [
    'enabled' => false,
    'limit'   => 100, // Number of requests
    'window'  => (60 * 15) // In seconds (15 minutes)
  ],

  /**
   * Configuration for storage
   * (rateLimit requires a storage path)
   * Path: A writable directory somewhere on your filesystem.
   */
  'storage' => [
    'path'  => false
  ]

];
```

## Example of Usage

```
use CoolApi\Instance;

// Instantiate Cool Api
$api = new Instance($config);

// Setup home route
$api->router->get('/', function ($req, $res) {

  // Return an output
  $res->status(200)->output([
    'foo' => 'bar'
  ]);
});

// Run the API
$api->run();
```

## Routing

To add a route for CoolApi, it's as simple as the following:

```
$api->router->get('/test', function ($req, $res) { /** Code here **/ });
```

And is the same for POST, PUT, or DELETE

```
$api->router->post('/test', function ($req, $res) { /** Code here **/ });
$api->router->put('/test', function ($req, $res) { /** Code here **/ });
$api->router->delete('/test', function ($req, $res) { /** Code here **/ });
```

Using parameters in the route itself:

```
$api->router->post('/user/:id', function ($req, $res) {

  // You can now access the id parameter via:
  var_dump($req->param('id'));

});
```

If the parameter does not exist, it will return false.

To get all parameters from the request: `$req->params`.

## `$api->router->use()`

The `use()` method allows you to use an array of routes that can be imported from other files to organize your code in a simple way. Below is an example of how to define a route in another file, and export it.

#### userRoutes.php

```
$data = function ($req, $res) {
  $id = $req->param('id');

  $res->output([
    'id' => $id
  ]);
};

return [
  ':id/data' => [
    'method'  => 'get',
    'handler' => $data
  ]
];
```


#### index.php

Below is how you would import userRoutes.php

```
$userRoutes = require_once __DIR__ . '/userRoutes.php';

// Make use of routes from other files
$api->router->use('/user', $userRoutes);

// Or with middleware:
$api->router->use('/user', $middleware, $userRoutes);
```

Now you can access `/user/:id/data` as a GET route.

## Use of $req

Getting POST, or GET variables:

```
$req->post('var_name'); // Returns $_POST['var_name'] || false
$req->get('var_name'); // Returns $_GET['var_name'] || false
$req->post() // Gets all $_POST variables
$req->get() // Gets all $_GET variables
```

Getting parameters from the URL:

```
// Returns false if it doesn't exist.
$req->param('id')

// Gets all parameters in object form
$req->params;
```

Getting headers from the request

```
$req->headers(); // Returns all headers in array form
```

Getting a specific header:

```
$req->header('User-Agent');
```

## Use of $res

Returning a normal response:

```
$res->output([
  'foo' => 'bar'
]); // Treats it as a normal status of 200, and outputs as JSON.
```

Setting the status:

```
$res->status(400)->output([]);
```

Setting the content type:

```
$res->status(200)->contentType('plain')->output('plain text');
$res->status(200)->contentType('html')->output('<html></html>');
```

## Enabling Cors Layer

```
[
  /**
   * Configuration for cors
   */
  'cors'    => [
    'enabled'   => true,
    'whitelist' => '*',
    'blacklist' => false
  ],
]
```

Above will allow all sites to access your api.

```
[
  /**
   * Configuration for cors
   */
  'cors'    => [
    'enabled'   => true,
    'whitelist' => [
      'https://www.google.com'
    ],
    'blacklist' => false
  ],
]
```

Above only allows requests from google.com

```
[
  /**
   * Configuration for cors
   */
  'cors'    => [
    'enabled'   => true,
    'whitelist' => '*',
    'blacklist' => [
      'https://www.google.com'
    ]
  ],
]
```

Above allows requests from anywhere except google.com

## Requiring an API Key in the request

You can use the following configuration to require an API key during the request to lockdown your API.

```
[
  /**
   * Configuration for api keys
   */
  'apiKeys'    => [
    'enabled'  => false,
    'keyField' => 'key',
    'keys'     => [
      'thisisanapikey'
    ]
  ],
]
```

`keyField` is looked at only if the request does not include a Authorization: Bearar <Token> in the request. It will then look for a `?key=` in the url, or `$_POST['key']`.

### Setting an origin for a specific key:

```
[
  /**
   * Configuration for api keys
   */
  'apiKeys'    => [
    'enabled'  => false,
    'keyField' => 'key',
    'keys'     => [
      'thisisanapikey' => [
        'origin' => 'www.facebook.com'
      ]
    ]
  ],
]
```

Now the key `thisisanapikey` is only accessible from the origin `www.facebook.com`
## Using Middleware

This api is setup so you can use your own middleware in the routes. Below is an example:

```
$middleware = function ($req, $res) {
  if (!$req->header('User-Agent')) {
    $res->status(400)->output([
      'error' => true,
      'message' => 'You must use a browser'
    ]);
  }

  return true; // Returning true, or returning nothing at all will pass.
}

/**
 * Passing $middleware as the 2nd argument tells the api
 * this needs to be ran before the handler method. If the middleware
 * returns a bad status, or returns false, the handler will never
 * be reached as the middleware fails.
 *
 * If the middleware returns true, then that means the handler can
 * successfully be reached and ran.
 */
$api->router->get('/test', $middleware, function ($req, $res) {
  // Do as you please here
});
```
