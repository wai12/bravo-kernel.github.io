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

- protect your API before even coding
- no code, no overhead
- json response consistent with the [FriendsOfCake API Listener](http://crud.readthedocs.org/en/latest/listeners/api.html#id1)
- manual API key updating and revocation
- replace with a "proper" solution when you're ready

## 1. Creating the nginx conf file

Create new file `/etc/nginx/api_keys.conf` with the following content:

```bash
# define boolean
set $valid 0;

# bravo-kernel's key
if ($http_apikey = 'abcdefg123456') {
    set $valid 1;
}

# mikey's key
if ($http_apikey = '1234567890') {
    set $valid 1;
}

# throw 403 if no key match was made
if ($valid = 0) {
    add_header 'Content-Type' 'application/json;charset=UTF-8' always;
    return 403 '{"success": false, "data":{"message":"Invalid API Key", "url": "$request_uri", "code":403}}';
}
```

The above will:
- will check all connections for the presence of an `apikey` header
- do nothing if a matching apikey is found
- will otherwise throw a 403 with json response body identical to
the [FoC API Listener](http://crud.readthedocs.org/en/latest/listeners/api.html#id1)

> Don't use underscores in your custom header (key) names [as nginx
> will remove them by default](http://stackoverflow.com/questions/22856136/why-underscores-are-forbidden-in-http-header-names).

## 2. Enabling API key protection

To enable API key protection for your virtual host:

1. Open your nginx vhost file

2. To protect all files (php, html, etc) add the following line directly below the `/` location

    ```
    include /etc/nginx/api_keys.conf;
    ```

    So your configuration will look similar to:

    ```nginx
    location / {
        include /etc/nginx/api_keys.conf;        
        try_files $uri \$uri /index.php?$args;
    }
    ```

4. Restart nginx to effectuate the changes:

    ```
    sudo service nginx restart
    ```

## 3. Testing your setup

1. Open a configurable REST client like [the Postman plugin](https://www.getpostman.com/) for Chrome.

2. Try to access one of your API resources

3. You should see a json (403) error response similar to the one below proving:

    - your API resources are no longer accessible without a valid API key
    - Nginx is serving json similar to the CRUD plugin

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
