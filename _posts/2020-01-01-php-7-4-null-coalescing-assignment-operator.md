---
layout: post
title:  "PHP 7.4: Null Coalescing Assignment"
author: michael
categories: [ development ]
thumb: assets/images/posts/codular-posts.jpg
comments: false
excerpt: "A brief look at one of the new features arriving in PHP 7.4, the ability to use the null coalescing operator on assignment."
hidden: true
---

#### Preface

With PHP 7.4 upcoming, it's time to start exploring some of the new features that will be arriving alongside it. Here we cover the enhancements around the _null coalescing operator_, namely the introduction of the null coalescing assignment operator. (Sometimes referred to as the "null coalesce equal operator")


#### Basics

In PHP 7 this was originally released, allowing a developer to simplify an `isset()` check combined with a ternary operator. For example, before PHP 7, we might have this code:

    $data['username'] = (isset($data['username']) ? $data['username'] : 'guest');

When PHP 7 was released, we got the ability to instead write this as:

    $data['username'] = $data['username'] ?? 'guest';

Now, however, when PHP 7.4 gets released, this can be simplified even further into: 

    $data['username'] ??= 'guest';


One case where this doesn't work is if you're looking to assign a value to a different variable, so you'd be unable to use this new option. As such, while this is welcomed there might be a few limited use cases. 

So for example, this code from before PHP 7 could only be optimised once using the null coalescing operator, and not the assignment operator:

    $username = (isset($_SESSION['username']) ? $_SESSION['username'] : 'guest');

becomes&hellip;

    $username = $_SESSION['username'] ?? 'guest';

<div class='post-footer-note'>
Note: This post was originally posted on Codular.com, but has since been migrated over.
</div>