---
title: Scraping a Ghost blog with PhantomJS and SlimerJS using CasperJS
date: 2017-06-03 02:19:49
categories:
  - JavaScript
tags:
  - Ghost
  - PhantomJS
  - SlimerJS
  - CasperJS
  - ES5
thumbnail: /images/javascript.png
---

This blog post will haunt your computer forever. You have been warned! :D

In this article, we are going to present different techniques to scrape a **Ghost** blog (especially with the default **Casper** theme). We will use **CasperJS**, which is a scripting utility for **PhantomJS** (WebKit headless browser) and **SlimerJS** (Gecko minimalist GUI browser).

<!-- more -->

## Installation

1. [PhantomJS](http://phantomjs.org/download.html): `npm i -g phantomjs-prebuilt`
2. [SlimerJS](https://slimerjs.org/download.html): `npm i -g slimerjs`
3. [CasperJS](http://docs.casperjs.org/en/latest/installation.html): `npm i -g casperjs`

## The basics

A regular CasperJS script is composed of **steps** and has the following structure:

```JavaScript
var casper = require('casper').create();

casper.start('http://www.website.com');

casper.then(function () {
  // Do something...
});

casper.run();
```

[`Casper.prototype`](http://docs.casperjs.org/en/latest/modules/casper.html#casper-prototype) defines lots of useful methods to interact with webpages and the most powerful one is probably [`evaluate()`](http://docs.casperjs.org/en/latest/modules/casper.html#evaluate) that gives a direct access to the DOM...

## Printing a list of the latest articles to console

OK, so... Let's take my personal blog in French as an example. In the following script (*ghost.js*), we get a list of the latest articles (these ones are displayed on the homepage):

```JavaScript
var casper = require('casper').create();

// Tell CasperJS to visit the homepage
casper.start('http://blog.badacadabra.net');

// Wait for articles to be loaded
casper.waitForSelector('article', function () {
  this.echo('BLOG TITLE:\n' + this.getTitle());
  this.echo('\nLATEST ARTICLES:');

  // Each title in the "titles" array returned by "evaluate"...
  this.each(this.evaluate(function () {
    var articles = document.querySelectorAll('article'),
        titles = [];

    for (var i = 0; i < articles.length; i++) {
      titles.push(articles[i].querySelector('.post-title').textContent);
    }

    return titles;
  }), function (self, title) {
    // ... should be displayed in console!
    this.echo(title);
  });
});

casper.run();
```

To run the script:

- PhantomJS: `casperjs ghost.js`
- SlimerJS: `casperjs --engine=slimerjs ghost.js`

Here is the output:

```
BLOG TITLE:
Hocus Pocus Numericus | Blog de Baptiste Vannesson (Badacadabra)

LATEST ARTICLES:
Les robots ont-ils des droits ?
Stress, anxiété et phobies à l'ère du numérique
Le règne de l'autodidaxie
Les limites du transhumanisme
L'ambiguïté du référencement payant
```

So far, so good! But what if we want to get a full list of all articles?

## Printing a list of all published articles to console

An elegant solution to achieve this goal is to use **recursion**. Our scraper starts on the homepage, tests if there are older posts (using pagination as a reference), and goes from page to page thanks to an IIFE (Immediately-Invoked Function Expression) which calls itself.

```JavaScript
var casper = require('casper').create();

casper.start('http://blog.badacadabra.net');

casper.then(function () {
  this.echo('BLOG TITLE:\n' + this.getTitle());
  this.echo('\nLATEST ARTICLES:');
});

// Recursive IIFE
(function getPostTitles() {
  casper.waitForSelector('article', function () {
    this.each(this.evaluate(function () {
      var articles = document.querySelectorAll('article'),
          titles = [];

      for (var i = 0; i < articles.length; i++) {
        titles.push(articles[i].querySelector('.post-title').textContent);
      }

      return titles;
    }), function (self, title) {
      this.echo(title);
    });
  });

  casper.then(function () {
    // If the link to older posts is present in pagination, ...
    if (this.visible('.older-posts')) {
      // ... click it to go to the next page, ...
      this.click('.older-posts');
      // ... then run the same operations on this new page
      getPostTitles();
    }
  });
})();

casper.run();
```

Cool! Getting lists of headlines is funny, but is it really useful? RSS feed readers already do this for us without efforts. Would it not be more interesting to extract all articles and create appropriate files to store local copies automatically? Let's do this right now!

## Extracting all articles to save local copies in filesystem

Extracting all articles is not so easy without a clear methodology. Our script should:

1. Start on the homepage
2. Visit the first article
3. Extract this article
4. Go back to the homepage
5. Visit the second article
6. ...
7. Go to the next page
8. Visit the first article of this page
9. ...
10. End when the last article from the last page has been extracted

It would be easy to define an array of predefined URLs, but this is not what we are going to do. This solution is not viable because each new article would not be taken into account by our script. In fact, we have to simulate the behavior of a user who would click each article in sequence to download all blog posts.

Here we will need to interact with the filesystem, so the **fs** module is required. For this script, we keep the recursive IIFE:

```JavaScript
var casper = require('casper').create(),
    fs = require('fs');

casper.start('http://blog.badacadabra.net');

var index = 1, // index of the article in the current page
    content = ''; // textual content of the latest extracted article

(function getPostContent() {
  // Wait for articles to be loaded
  casper.waitForSelector('article', function () {
    // Click the first link (the index starts at 1)
    this.click('.post:nth-of-type(' + index + ') a');
  });

  casper.then(function () {
    // Extract the textual content of the article
    content = this.evaluate(function () {
      return document.querySelector('.post').textContent;
    });
  });

  casper.then(function () {
    var title = this.getTitle();
    this.echo('EXTRACTING - ' + title);
    // Process the title string to create a decent filename
    title = title.replace(/[ '",]/g, '-').toLowerCase();
    // Write the file to disk
    fs.write('articles/' + title, content, 'w');
    // Go back to the homepage
    this.back();
  });

  casper.waitForSelector('article', function () {
    index++;
    // If there is another article on this page, get its content
    if (this.visible('.post:nth-of-type(' + index + ')')) {
      getPostContent();
    } else { // If not, continue on the next page (if there is one)
      if (this.visible('.older-posts')) {
        index = 1;
        this.click('.older-posts');
        getPostContent();
      }
    }
  });
})();

casper.run();
```

Done! This script will create a local `articles/` directory containing articles/files with raw textual content.

## Conclusion

CasperJS is a powerful automation library on top of PhantomJS and SlimerJS. Moreover, thanks to a clean HTML code, a Ghost blog is a pleasure to scrape!

If you want to learn more about these amazing technologies, maybe you can try to scrape articles filtered by author, by tag or by date... This is not very difficult with the code provided here, but this is a good exercise. You may also be interested in the [Ghost API](https://api.ghost.org/v0.1/docs).

Spectres have never been so kind!
