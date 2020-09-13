---
layout: post
title:  "PHP 7.4: Typed Properties"
author: michael
categories: [ development ]
thumb: assets/images/posts/codular-posts.jpg
comments: false
excerpt: "Diving into another upcoming feature in PHP 7.4, this time looking at the addition of type casting on properties of classes."
hidden: true
---

#### Preface

With PHP 7.4 upcoming, it's time to start exploring some of the new features that will be arriving alongside it. In this article we look into the new addition of **Typed Properties**, this is where properties on objects can be type cast instead. This is a key change to help push PHP along on the type path, enforcing things such as type-safety.


#### Basics

The best example for this is the example that's actually provided within the [RFC itself](https://wiki.php.net/rfc/typed_properties_v2). The below chunk of code is what you might need pre-7.4 to ensure that the values that you're passing in etc match the type of the properties.

```php
class User {
    /** @var int $id */
    private $id;
    /** @var string $name */
    private $name;
 
    public function __construct(int $id, string $name) {
        $this->id = $id;
        $this->name = $name;
    }
 
    public function getId(): int {
        return $this->id;
    }
    public function setId(int $id): void {
        $this->id = $id;
    }
 
    public function getName(): string {
        return $this->name;
    }
    public function setName(string $name): void {
        $this->name = $name;
    }
}
```

This can be simply rewritten as: 

```php
class User {
    public int $id;
    public string $name;
 
    public function __construct(int $id, string $name) {
        $this->id = $id;
        $this->name = $name;
    }
}
```

The reason a lot of this can be simplified, is because in the original code, you had to use the `setId` or `setName` methods to ensure that the values passed in were of the correct type. In PHP 7.4, you can now go directly to the parameter and PHP itself will ensure that the types of the values you assign to the properties match those the class defintion requires.


<div class='post-footer-note'>
Note: This post was originally posted on Codular.com, but has since been migrated over.
</div>