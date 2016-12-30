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
- json error response consistent with the [FriendsOfCake API Listener](http://crud.readthedocs.org/en/latest/listeners/api.html#id1) or JSONAPI
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

# throw 403 with CakePHP Crud compatible JSON error response if no key match was made
if ($valid = 0) {
    add_header 'Content-Type' 'application/json;charset=UTF-8' always;
    return 403 '{"success": false, "data":{"message":"Invalid API Key", "url": "$request_uri", "code":403}}';
}
```

If you prefer JSONAPI simply replace the `$valid` condition above with:

```bash
 # Throw 403 with JSONAPI error response if no key match was made
if ($valid = 0) {
	add_header 'Content-Type' 'application/vnd.api+json;charset=UTF-8' always;
	add_header 'Access-Control-Allow-Origin' '*' always;
	return 403 '{"errors": [{ "status": "403", "source": { "pointer": "$request_uri" }, "title":  "Forbidden", "detail": "Invalid API key" }]}';
}
```

The above:
- will check all connections for the presence of an `apikey` header
- will do nothing if a matching apikey is found, otherwise:
	- throws a 403 error
	- produces a JSON  error response body in either JSONAPI or 
	[FoC API Listener](http://crud.readthedocs.org/en/latest/listeners/api.html#id1)
	compatible format

> Do NOT use underscores in your custom header (key) names [as nginx
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

    > Please note that will ONLY protect your php served files. To protect
    > e.g. static html folders simply include the `api_keys.conf` in those
    > locations as well. 

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
