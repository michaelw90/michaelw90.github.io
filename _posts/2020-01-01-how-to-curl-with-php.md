---
layout: post
title:  "Getting Started with cURL Requests in PHP"
author: michael
categories: [ development ]
thumb: assets/images/posts/curl-with-php.jpg
comments: false
excerpt: "A very simple introduction to GET and POST requests with cURL in PHP."
---

#### Preface

cURL allows transfer of data across a wide variety of protocols, and is a very powerful system. It's widely used as a way to send data across websites. cURL is unrestricted in what it can do, from the basic HTTP request, to the more complex FTP upload or interaction with an authentication enclosed HTTPS site. We'll be looking at the simple difference between sending a `GET` and `POST` request and dealing with the returned response, as well as highlighting some useful parameters.

#### Basics

Before we can do anything with a cURL request, we need to first instantiate an instance of cURL - we can do this by calling the function `curl_init();`, which returns a cURL resource. This function takes one parameter which is the URL that you want to send the request to, however, in our case, we'll hold off doing that for now and set it an alternatively way later.

##### Settings

Once we've got a cURL resource, we can begin to assign some settings, below is a list of some of the core ones that I set

- `CURLOPT_RETURNTRANSFER` - Return the response as a string instead of outputting it to the screen
- `CURLOPT_CONNECTTIMEOUT` - Number of seconds to spend attempting to connect
- `CURLOPT_TIMEOUT` - Number of seconds to allow cURL to execute
- `CURLOPT_USERAGENT` - Useragent string to use for request
- `CURLOPT_URL` - URL to send request to
- `CURLOPT_POST` - Send request as `POST`
- `CURLOPT_POSTFIELDS` - Array of data to POST in request

We can set a setting by using the `curl_setopt()` method, which takes three parameters, the cURL resource, the setting and the value. So, to set the URL that we're sending the request to as `http://testcURL.com` we use the following code:

```php
$curl = curl_init();
curl_setopt($curl, CURLOPT_URL, 'http://testcURL.com');
```

As mentioned, we can set the URL by sending a parameter through when getting the cURL resource:

```php
$curl = curl_init('http://testcURL.com');
```

It is possible to set multiple settings at one time by passing through an array of settings and values to the function `curl_setopt_array()`:

```php
$curl = curl_init();
curl_setopt_array($curl, [
    CURLOPT_RETURNTRANSFER => 1,
    CURLOPT_URL => 'http://testcURL.com'
]);
```

##### Sending request

When all of the options are sent, and the request is ready to send, we can call the `curl_exec()` method which will execute the cURL request. This function can return three different things:

- `false` - if there is an error executing the request
- `true` - if the request executed without error and `CURLOPT_RETURNTRANSFER` is set to `false`
- *The result* - if the request executed without error and `CURLOPT_RETURNTRANSFER` is set to `true`

Using the previous example, where we are wanting to get the result back, we would use the following: 

```php
$result = curl_exec($curl);
```

With `$result` now containing the response from the page - which might be JSON, a string or a full blown site's HTML.

### Close Request

When you've sent a request and got the result back, you should look to close the cURL request so that you can free up some system resources, this is as simple as calling the `curl_close()` method which as with all other functions takes the resource as its parameter.

### GET Request

A `GET` request is the default request method that is used, and is very straight forward to use, infact all of the examples so far have been `GET` requests. If you want to send parameters along in the request you simply append them to the URL as a query string such as `http://testcURL.com/?item1=value&item2=value2`.

So for example to send a `GET` request to the above URL and return the result we would use: 
    
```php
// Get cURL resource
$curl = curl_init();
// Set some options - we are passing in a useragent too here
curl_setopt_array($curl, [
    CURLOPT_RETURNTRANSFER => 1,
    CURLOPT_URL => 'http://testcURL.com/?item1=value&item2=value2',
    CURLOPT_USERAGENT => 'Codular Sample cURL Request'
]);
// Send the request & save response to $resp
$resp = curl_exec($curl);
// Close request to clear up some resources
curl_close($curl);
```

#### POST Request

The sole difference between the `POST` and `GET` request syntax is the addition of one setting, two if you want to send some data. We'll be setting `CURLOPT_POST` to `true` and sending an array of data through with the setting `CURLOPT_POSTFIELDS`

So for example switching the above `GET` request to be a `POST` request, we would use the following code:

```php
// Get cURL resource
$curl = curl_init();
// Set some options - we are passing in a useragent too here
curl_setopt_array($curl, [
    CURLOPT_RETURNTRANSFER => 1,
    CURLOPT_URL => 'http://testcURL.com',
    CURLOPT_USERAGENT => 'Codular Sample cURL Request',
    CURLOPT_POST => 1,
    CURLOPT_POSTFIELDS => [
        item1 => 'value',
        item2 => 'value2'
    ]
]);
// Send the request & save response to $resp
$resp = curl_exec($curl);
// Close request to clear up some resources
curl_close($curl);
```

There you have a `POST` request that will work the same as our `GET` request above and return the response back to the script so that you can use it as you want.


#### Errors

As much as we all hate errors, you really need to take care to account for any eventuality with cURL as ultimately you will not have control over the site(s) that you are sending your request to, you cannot guarantee that the response will be in the format that you want, or that the site will even be available.

There are a few functions that you can use to handle errors and these are:

- `curl_error()` - returns a string error message, will be blank `''` if the request doesn't fail.
- `curl_errno()` - which will return the cURL error number which you can then look up on [this page listing error codes](http://curl.haxx.se/libcurl/c/libcurl-errors.html).

An example usage would be: 

```php
if (!curl_exec($curl)) {
    die('Error: "' . curl_error($curl) . '" - Code: ' . curl_errno($curl));
}
```

You might want to look at using the setting `CURLOPT_FAILONERROR` as `true` if you want any HTTP response code greater than 400 to cause an error, instead of returning the page HTML.


#### curl_exec($theEnd);

cURL is a behemoth, and has many many possibilities. Some sites might only serve pages to some user agents, and when working with APIs, some might request you send a specfici user agent, this is something to be aware of. If you want to have a play with some cURL requests, why not have a go at playing with [oAuth with Instagram](http://codular.com/oauth-authentication-with-instagram). 
<div class='post-footer-note'>
Note: This post was originally posted on Codular.com, but has since been migrated over.
</div>