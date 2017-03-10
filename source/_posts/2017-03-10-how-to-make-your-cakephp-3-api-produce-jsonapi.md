title: "How to make your CakePHP 3 API produce JSON API"
date: 2017-03-10 20:10:24
categories:
- CakePHP
tags:
- cakephp
- cakephp3
- rest
- api
- json api
---

In this follow-up post to
[How to add JWT Authentication to a CakePHP 3 REST API](/2015/04/how-to-add-jwt-authentication-to-a-cakephp-3-rest-api/)
we will update our existing API so it will adhere to the
[JSON API specification](http://jsonapi.org/) giving you the
benefits of standardization and instant compatibility with JSON API supporting tools and frameworks
like [Ember Data's JSONAPIAdapter](https://github.com/emberjs/data#a-brief-note-on-adapters).


[> API methods used in this post shared with Postman](https://app.getpostman.com/run-collection/197398a609a6d233a8c2)

## Before We Begin

This is part five of the CakePHP 3 REST API tutorial series:

1. [How to build a CakePHP 3 REST API in minutes](/2015/04/how-to-build-a-cakephp-3-rest-api-in-minutes/)
2. [How to use a CakePHP 3 REST API](http://cakebox:4000/2015/04/how-to-use-a-cakephp-3-rest-api/)
3. [How to prefix route a CakePHP 3 REST API](/2015/04/how-to-prefix-route-a-cakephp-3-rest-api/)
4. [How to add JWT Authentication to a CakePHP 3 REST API](/2015/04/how-to-add-jwt-authentication-to-a-cakephp-3-rest-api/)
5. How to make your CakePHP 3 API produce JSON API

Before starting this tutorial either:

+ complete the previous tutorial
+ start fresh by using these
[end-state application sources](https://github.com/bravo-kernel/application-examples/tree/master/blog-how-to-make-your-cakephp-3-api-produce-jsonapi),
composer installing and running the database migration

## 1. Install Neomerx/jsonapi

The Crud JsonApiListener used to produce the JSON API
depends on the [neomerx/jsonapi](https://github.com/neomerx/json-api)
package so install that first by running:

```bash
composer require neomerx/json-api:^0.8.10
```

## 2. Upgrade project packages

To make sure your API is using an up-to-date version of CakePHP and the 
required version of the Crud plugin now update your project's
composer packages by running:

```bash
composer update
```

> Please note that the Crud JsonApiListener is not available in Crud releases prior to 4.4.0.

## 3. Disable JWT authentication

Since JWT authentication is unrelated to JSON API we will disable it by 
removing the related lines in `src/Controller/Api/AppController`
which should leave you with a file looking like this:

```php
<?php
namespace App\Controller\Api;

use Cake\Controller\Controller;
use Cake\Event\Event;

/**
 * AppController specific to API resources
 */
class AppController extends Controller
{

    use \Crud\Controller\ControllerTrait;

    public function initialize()
    {
        parent::initialize();

        $this->loadComponent('RequestHandler');
        $this->loadComponent('Crud.Crud', [
            'actions' => [
                'Crud.Index',
                'Crud.View',
                'Crud.Add',
                'Crud.Edit',
                'Crud.Delete'
            ],
            'listeners' => [
                'Crud.Api',
                'Crud.ApiPagination',
                'Crud.ApiQueryLog'
            ]
        ]);
    }
}
```

> Even though the CakePHP and Crud upgrades were **totally
> transparent** while writing this post this might be a good
> moment to verify that your API's `cocktails` endpoint is
> still producing the expected results using GET, POST, etc.

## 3. Enable JSON API

All that is needed now to make your API produce JSON API is opening
`src/Controller/Api/AppController` and replacing the `Crud.Api`
listener with `Crud.JsonAPi` so it looks like:

```php
            'listeners' => [
                'Crud.JsonApi',
                'Crud.ApiPagination',
                'Crud.ApiQueryLog'
            ]
```

All done, seriously.

## 4. Using the new API

The JsonApiListener documentation contains very detailed
[usage descriptions](http://crud.readthedocs.io/en/latest/listeners/jsonapi.html#response-formats)
and since there no point in duplicating them here we will
provide you with just one example to get you going.

**Never forget:**

- ALL requests to your new API MUST use the `application/vnd.api+json` Accept Header
- ALL requests with post data MUST use the `application/vnd.api+json` Content-Type Header

### View action (GET)

For this example we are in the mood for a Mojito so we query 
``http://cake3api.app/cocktails/3`` making sure we are using:

- **HTTP Method** ``GET``
- **Accept Header** ``application/vnd.api+json``

{% asset_img api-request-headers-view.png 'API Request Headers for view action' %}

If things went well your API should return Status Code 200 (Success) with a response body in
JSON API format looking similar to:

```json
{
  "data": {
    "type": "cocktails",
    "id": "3",
    "attributes": {
      "name": "Mojito",
      "description": "Rum based",
      "created": "2015-04-11T09:52:01+00:00",
      "modified": null
    },
    "links": {
      "self": "/api/cocktails/3"
    }
  }
}
```

> See the Postman collection (top of page) for examples of `index`, `add`, `edit` and `delete`.

## 5. Make it better

Even though the JsonApiListener is already quite feature-complete, some parts of the JSON API specification 
(like [Sparse Fieldsets](http://jsonapi.org/format/#fetching-sparse-fieldsets) and query parameters) have
not been implemented yet. Feel free to submit a PR for missing functionality and help work towards a
full-featured implementation of the specification, the effort should be minimal.

## Additional reading

- [Crud JsonApiListener documentation](http://crud.readthedocs.io/en/latest/listeners/jsonapi.html)
- [Git repository](https://github.com/bravo-kernel/application-examples/tree/master/blog-how-to-make-your-cakephp-3-api-produce-jsonapi) with working end state application as produced by this tutorial
- [neomerx/jsonapi](https://github.com/neomerx/json-api)

<em>Hat tip to [josbeir](https://github.com/josbeir) for the inspiration found in his plugin.</em>
