## Laravel Sail HTTPS with Caddy

This project is a demonstration of how to use Laravel Sail over HTTPS by using Caddy's reverse proxy ability.

This project also uses Laravel breeze for basic web authentication. You can use redis to store session information
as seen in the .env.example.

## Docker setup

### Introduction

The docker-compose.yml file has images for sail, caddy, redis and mysql by default.

Since docker maps the container's name to its ip, `laravel.test` points to the ip of the
laravel.test image, which is `localhost` by default.

Whilst the docker containers are running, you should only run php and artisan commands inside the laravel.test container.

### Configuration

You can add any other images you may want by running `php artisan sail:add`.

The php.ini file used in the laravel.test container will depend on which version of sail's image you are using in the
docker-compose.yml. The default version is `sail-8.3/app` and so the php.ini file can be found at ./docker/8.3/php.ini.

To change which version of sail's image you use, make sure you change the both the 'image' and 'context' sections of the
sail image in `docker-compose.yml`.

When you have configured the docker-compose.yml file, run `php ./vendor/bin/sail up` to build the images.

You may use the container's name instead of its ip in the .env file as well. For example, for the REDIS_HOST variable,
you can use `redis` instead of `127.0.0.1`.

## Caddy setup

### Introduction

The caddy image will set up a caddy server to act as a proxy and self-assign ssl certificates, so you are able to access
localhost via https. Caddy will also forward any requests you make from http to https.

As seen in the Caddyfile, caddy will make a request to http://laravel.test/caddy to see if a domain is authorized to be
issue a ssl certificate. Since docker maps the container's name to its ip, `laravel.test` points to the ip of the
laravel.test image, which is `localhost` by default.

The caddy endpoint currently only returns a 200 response in the `web.php` file but you may
replace this with your own authorization checks.

### Configuration

To change the ports that Caddy listens to, you must change the `ports` section of the `caddy` image in the
`docker-compose.yml` file, and you must also change the ports inside the `Caddyfile` file. You must then run
`php ./vendor/bin/sail up` again to rebuild the images.

By default, these ports are `443` and `80` for https and http respectively.

### Notes

- When submitting forms and making requests from a page which require a csrf token, make sure the url starts with 'https'.
Not doing this will most likely result in a 419 error. You can solve this by making sure the url / action of the form 
and requests start with https, and when using functions like `redirect`, make sure the `secure` parameter is `true` to 
force https. You should also set your `APP_URL` in your env to `localhost`


- Browsers will tell you that 'this site is not secure'. If you are using Google Chrome, you can either click `Advanced`
and `proceed to...` or click anywhere in the browser window and type 'thisisunsafe' (not copy and paste) to continue


- It is a hacky solution, but by adding `URL::forceScheme('https');` to the `boot` method of `AppServiceProvider`, functions such as
`route()` will use https. (The namespace for the URL Facade class is `Illuminate\Support\Facades\URL`).
