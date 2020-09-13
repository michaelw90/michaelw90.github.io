---
layout: post
title:  "iOS Push Notifications with PHP"
author: michael
categories: [ development ]
thumb: assets/images/posts/codular-posts.jpg
comments: false
excerpt: "Nearly every site that you visit nowadays has some sort of database storage in place, many sites opt to use MySQL databases when using PHP. But are you interacting with your databases correctly?"
hidden: true
---

#### Introduction

Sending notifications to an iOS Application can enhance the experience exponentially for users, also it allows you to deliver key data easily. However, actually sending the push notification to users can be a bit tedious at times, and at times confusing. You need to ensure that you pack your integers, and times correctly - failing to do this and you'll probably get an unhelpful status from Apple. 

#### The APNS

Using the APNS (Apple Push Notification Service) requires a bit of pre-configuration initially, you'll need to download some certificates from the server initially, as these are used for the connections. 

Next, you'll need to make sure you understand what it is that we're going to be sending, from the [Apple Website](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/CommunicatingWIthAPS.html).

This highlights the notification that we're going to be sending through to Apple. To break it down, and make sense of it, we're going to be sending a series of *"items"* which are: 

1. Device Token
2. Payload
3. Notification Identifier
4. Expiration Date
5. Priority

Each of the items that we send is going to have an ID, the length & the data. This is getting complicated, let's look at some code...

#### Constructing the Notification

Let's go through each of the various items we have to include in our notification first, we'll then look to construct the rest of the notification afterwards. We're going to be making extensive use of the (http://php.net/manual/en/function.pack.php)[Pack] method here, be sure to give it a glance.

##### Device Token

With id of **1**, this is required to be a length of **32 bytes** only, let's assume that we have a device token in a variale `$deviceToken`, and we'll go from there: 

```php
// Id of 1
chr(1)
// The length (32 bytes)
. pack('n', 32)
// Hex String of Device Token
. pack('H*', $deviceToken);
```

##### Payload

With id of **2**, this is the payload that we're sending, so before we can actually build this, we need to identify the payload. We are just going to send a very basic notification with the string "Hello World", and we're going to set the badge count to **1** as well - this needs to be JSON as well - so don't forget your `json_encode()` function here. 

```php
$payload = json_encode([
    'aps' => [
        'alert' => 'Hello World',
        'sound' => 'default',
        'badge' => 1
    ]
]);
```

One thing to note here, is that we're using the default sound, you are able to change this sound to either another stock sound on the device, or you can reference a sound that has been included in the application. 

Now that we have the payload defined, we are able to start defining the next part of our notification string: 

```php
// Id of 2
chr(2)
// Length of the payload
. pack('n', strlen($payload))
// Payload that we're sending
. $payload
```

##### Notification Identifier

With id of **3**, the notification identifier needs to just be unique so that you can identify the notification if it fails to send, or needs resending etc. If you pass notifications in with the same ID, they may be overwritten, instead of all delivered. As we're just sending one notification, we can just set this to a single number (1), however in a system you'd probably have your notifications to send in a database, and use the unique identifier from that table. 

As defined by Apple's rules, you need to make sure that this has a length of 4, this doesn't mean that the id has to be 1000+, it just means we have to pack it correctly...

```php
// Id of 3
chr(3)
// Length of integer is 4
. pack('n', 4)
// Pack notifier to length of 4
. pack('N', $token->id)
```

##### Expiration Date

With id of **4**, we just specify the expiration date for the notification, this is used to idenfity when the notification is no longer valid, and so that it can be discarded. If you have time specific information (ie relating to something that expires is a set time), you will want to set this expiration date to that. So that you're not sending out of date notifications to people. 

```php
// Id of 4
chr(4)
// Length of 4
. pack('n', 4)
// Set expiration to 1 day from now
. pack('N', time() + 86400)
```

##### Priority

With id of **5**, this identifies the priority that the notification is sent with. In most cases, this will be set to 10, but you have the option to send a value of 5 which will send at a time that conserves the power on the device. 

```php
// Id of 5
chr(5)
// Length of 1
. pack('n', 1)
// Send immediately
. chr(10);
```

#### Remaining Notification Construction

With the items constructed, we now need to define the rest of the notification, as part of this is based off the length of the items. So first we need to construct the "inner" section, and then construct the rest of the notification.

```php
$inner = 
      chr(1)
    . pack('n', 32)
    . pack('H*', $token->value)

    . chr(2)
    . pack('n', strlen($payload))
    . $payload

    . chr(3)
    . pack('n', 4)
    . pack('N', $token->id)

    . chr(4)
    . pack('n', 4)
    . pack('N', time() + 86400)

    . chr(5)
    . pack('n', 1)
    . chr(10);
```

With that, we can construct the rest, which is a command value of *2*, followed by the length of the `$inner` and then the actual `$inner` value: 

```php
$notification = 
      chr(2)
    . pack('N', strlen($inner))
    . $inner;
```

Now we'll be sending, and using `$notification` to send to Apple - to send we're going to streaming the notifications, instead of sending via Curl. 

#### Sending Notification

We need to connect to the APNS, identify our certificate (along with passphrase), then start to stream. The URL that we stream to depends on whether we are using production, or not. These URLs are at the start of the block. Below is some commented code that will do that for you, hopefully it's clear what is going on:

```php
if ($production) {
    $gateway = 'gateway.push.apple.com:2195';
} else { 
    $gateway = 'gateway.sandbox.push.apple.com:2195';
}

// Create a Stream
$ctx = stream_context_create();
// Define the certificate to use 
stream_context_set_option($ctx, 'ssl', 'local_cert', '/certificates/mycert.pem');
// Passphrase to the certificate
stream_context_set_option($ctx, 'ssl', 'passphrase', 'codular-test');

// Open a connection to the APNS server
$fp = stream_socket_client(
    $gateway, $err,
    $errstr, 60, STREAM_CLIENT_CONNECT|STREAM_CLIENT_PERSISTENT, $ctx);

// Check that we've connected
if (!$fp) {
    die("Failed to connect: $err $errstr");
}

// Ensure that blocking is disabled
stream_set_blocking($fp, 0);

// Send it to the server
$result = fwrite($fp, $notification, strlen($notification));

// Close the connection to the server
    fclose($fp);    
```

#### Conclusion

What we have is a very basic method for constructing, and sending push notifications to Apple's Push Notification Service. One thing to consider here is that Apple suggest that you periodically poll their feedback service which will inform you of any failed notifications. 

The thinking here is that it will identify devices that have perhaps uninstalled the application, or no longer exist and so you should stop sending push notifications to them. 

More information can be found at the bottom of the [aforementioned page](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/CommunicatingWIthAPS.html).


<div class='post-footer-note'>
Note: This post was originally posted on Codular.com, but has since been migrated over.
</div>