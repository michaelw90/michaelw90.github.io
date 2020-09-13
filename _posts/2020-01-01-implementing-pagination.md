---
layout: post
title:  "How to Implement Pagination"
author: michael
categories: [ development ]
thumb: assets/images/posts/codular-posts.jpg
comments: false
excerpt: "Implementing pagination can sometimes be a pain, mostly down to being confused by the logic required to work out total pages etc, it's all explained within."
hidden: true
---

#### Preface

Most of the issues people face when implementing pagination is not fully understanding the logic behind how it works, once that has been explained fully the actual coding is pretty straightforward.

Firstly, we'll start with an explanation of what we want to do, then guide through the basic code that we'll use to implement the pagination.

#### The plan

So we're planning on showing the page numbers for a series of pieces of data so that users can easily navigate multiple items. There are a few things that we need to know first: 

- The page that we're on
- Total number of items
- Number of items per page

Next we need to look at how we want to display the page numbers, there is a wide array of methods that people use: 

- Simple Next/Previous buttons with no numbers
- Page 1 & the last page, with the current page (and 2 above/below) shown
- A list of all possible pages

I personally prefer to show the very first page, that last page, and then the current page with 2 pages above & below. So for example on page 12 out of 24 pages we'd see: 

```
1, 10, 11, 12, 13, 14, 24
```

This allows users to quickly navigate to the start, and to the end, as well as jump through multiple pages at once.

##### The Arithmetic

First we need to work out the total number of pages, for this, we want to take the total number of items that there are, and divide it by the number of items per page. But we want to make sure that we take that number and round it up. 

So if there were 12 items in total, and we were showing 5 per page, we'd have a total of 3 pages of items. If we were to show 3 per page, we'd show 4 pages. 

Next we need to ensure that the page that we're on is within the bounds of the page numbers that we have, so if the user navigates to a page that is too high, we will show them the last page that is available - and also if the page is less than 1, we push the user to the first page.

#### Coding

For this, we're going to be using a really simple MySQL table (so that we can demonstrate what SQL to use) that is called `articles` and has 2 columns `title` and `id`. *Realistically the table will have more columns.* With the SQL that we're using we will be making extensive use of the `LIMIT` keyword within our queries, be sure to check out an [earlier article](http://codular.com/sql-introduction) that explains it.

##### Total Number of Articles

For this, we simply run the following SQL query to `COUNT` the number of rows in the table and then with some other PHP (omitted), we will store in a variable `$totalArticles`:

```php
SELECT COUNT(*) as `total` FROM `articles`;
```

With this total, we can now take the current page number and check that the page is in range of our max & minimum.

##### Total Number of Pages

Now we need the total number of pages, this will be displayed, so that we can check that the current page is no higher than this: 

```php
// Remember to round it up always!
$totalPages = ceil($totalArticles / $articlesPerPage);
```

##### Check Current Page

We'll be using a `GET` parameter called `page` for this, which we can get using `$_GET['page']`:

```php
// Check that the page number is set.
if (!isset($_GET['page'])) {
    $_GET['page'] = 0;
} else {
    // Convert the page number to an integer
    $_GET['page'] = (int)$_GET['page'];
}

// If the page number is less than 1, make it 1.
if ($_GET['page'] < 1) {
    $_GET['page'] = 1;
    // Check that the page is below the last page
} else if ($_GET['page'] > $totalPages) {
    $_GET['page'] = $totalPages;
}
```

Now that we've checked, and assigned the correct page number to the `GET` parameter, we can start using that as the current page. 


#### Listing the Pages

Now that we have the total number of pages, and the current page that the user is on, we need to list the pages. We'll simply loop through from 1 to the last page, but skip any page that isn't within 2 above or 2 below of the current page: 

```php
foreach (range(1, $totalPages) as $page) {
    if ($page == 1 || $page == $totalPages || ($page >= $_GET['page'] - 2 && $page <= $_GET['page'] + 2)) {
        echo '<a href="?page=' . $page . '">' . $page . '</a>';
    }
}
```

That will output all of the pages. But it might be that we want to draw attention to which page you're on at the moment in two ways, one is to remove the actual link, and the second is to give it a class - so if it's the current page, we'll put it in a `span` with a class of `currentpage`: 

```php
foreach (range(1, $totalPages) as $page) {
    // Check if we're on the current page in the loop
    if ($page == $_GET['page']) {
        echo '<span class="currentpage">' . $page . '</span>';
    } else if ($page == 1 || $page == $totalPages || ($page >= $_GET['page'] - 2 && $page <= $_GET['page'] + 2)) {
        echo '<a href="?page=' . $page . '">' . $page . '</a>';
    }
}
```

We're using the `range()` method here, this returns an array with numbers in the range that we've chosen. So in the above example we'll end up with an array that has all of the numbers from 1 up to the total number of pages. 


#### Pull out Articles

Finally, we need to pull out the pages that are to be shown on the page that the user is on - to do this we need to make use of the aforementioned `LIMIT` feature. 

We're going to need two numbers to use for the `LIMIT`, the first will be the number to start at (starting at 0), and the second will be the total to pull out. 

The **first** number can be found out by taking one off the current page number, and multiplying it by the total number per page, so for the first 3 pages we'd get (imagining 6 items per page): 

1. (1 - 1) * 6 = 0
2. (2 - 1) * 6 = 6
3. (3 - 1) * 6 = 12

So to throw it all together we'd end up with some PHP to build up a variable, then throw in that in the SQL that we will run: 

```php
// Calculate the starting number
$startArticle = ($_GET['page'] - 1) * $articlesPerPage;

// SQL Query
$sql = 'SELECT * FROM `articles` LIMIT ' . $startArticle . ', ' . $articlesPerPage;
```

With that query, it will automatically alter for the page that the user is on, and return the posts.


#### Conclusion

Pagination is brilliant, and actually a lot of the above methods would still stand if you were looking to use an infinite scrolling page instead. The main difference would be that you would AJAX to the second (or whatever) page to get the extra content, then append it to the current content.

The above is provided as is, there might be better methods for doing it, but of course a lot of the code that you'd use depends on how you want to show the list of pages to the visitors of the site.


<div class='post-footer-note'>
Note: This post was originally posted on Codular.com, but has since been migrated over.
</div>