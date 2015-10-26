title: Instant Nginx API key authentication for your CakePHP API
date: 2015-10-26 20:50:30
categories:
  - cakephp
tags:
  - nginx
  - cakephp
  - foc
  - api
---
Instantly add rock solid API key authentication for your CakePHP
([CRUD](https://github.com/FriendsOfCake/crud)) API using
nothing more than a simple Nginx configuration file:

- no need for 3rd party gateways
- no code, no overhead
- manual API key revocation
- manual API key updating
- json responses consistent with the [FriendsOfCake API Listener](http://crud.readthedocs.org/en/latest/listeners/api.html#id1)

## 1. Creating the nginx conf file

Create new file `/etc/nginx/api_keys.conf` with the following content:

```bash
# bravo-kernel's key
if ($http_apikey = 'abcdefg123456') {
    break;
}

# mikey's key
if ($http_apikey = '1234567890') {
    break;
}

default_type application/json;
return 403 '{"success": false, "data":{"message":"Invalid API Key", "url": "$request_uri", "code":403}}';
```

The above will:
- will check all connections for the presence of an `apikey` header
- if an apikey is found with a matching value no further action is taken
- if no match was made a 403 is thrown with json response body identical to
the [FoC API Listener](http://crud.readthedocs.org/en/latest/listeners/api.html#id1)

> Don't use underscores in your custom header (key) names [as nginx
> will remove them by default](http://stackoverflow.com/questions/22856136/why-underscores-are-forbidden-in-http-header-names).

## 2. Enabling API key protection

To enable API key protection for your virtual host:

1. Open your nginx vhost file

2. Add the following line directly below the `~ \.php` location

    ```
    include /etc/nginx/api_keys.conf;
    ```

    So your configuration will look similar to:

    ```nginx
    location ~ \.php$ {
        include /etc/nginx/api_keys.conf;        
        try_files $uri =404;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass    unix:/var/run/php5-fpm.sock;
        fastcgi_index   index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    ```

4. Restart nginx to effectuate the changes:

    ```
    sudo service nginx restart
    ```

## 3. Testing your setup

1. Open a configurable REST client like [the Postman plugin](https://www.getpostman.com/) for Chrome.

2. Try to access one of your API resources

3. You should see a json error response (403) similar to the one below proving
your API resources can no longer be accessed without a valid API key:

    ```
    {
      "success": false,
      "data": {
        "message": "Invalid API Key",
        "url": "/cocktails",
        "code": 403
      }
    }
    ```

4. Now add a custom header named `apikey` and value `1234567890`

5. Try accessing the same resource

6. If things went well you should a familiar CakePHP success response (200)
containing your API resources in the json body.
