---
title: 'End-to-End testing with Nightwatch.js: TDD vs BDD'
tags:
  - Nightwatch.js
  - Testing
  - Automation
  - TDD
  - BDD
  - Selenium
  - WebDriver
categories:
  - JavaScript
thumbnail: /images/javascript.png
date: 2017-06-13 23:43:15
---


Have you ever heard of **Nightwatch.js**, the well-known "*Node.js powered End-to-End testing framework*"? Here we will experiment with the two main testing components of Nightwatch, dealing with the classicism of **Test-Driven Development** (TDD) and the agility of **Behavior-Driven Development** (BDD).

To be consistent, let's test the [Nightwatch](http://nightwatchjs.org/) official website!

<!-- more -->

## Nightwatch.js: what for?

**End-to-End testing** is a key step in software development process. E2E tests are not the most complete ones because it is impossible to cover everything in an app at the top of the **testing pyramid**. E2E tests are generally slow and resource-intensive. However, they are richer than any other type of test because they bring all components together and consider the application as a whole. For this reason, E2E testing is often performed as **acceptance testing**.

Contrary to other popular testing solutions like **CasperJS** which works on top of **PhantomJS** or **SlimerJS**, Nightwatch has been designed to work with *real* web browsers using **Selenium/WebDriver**. Thanks to Nightwatch, it is possible to control, remotely and programmatically, true instances of Firefox or Chromium.

Nightwatch testing API is based on two testing components: **Assert** and **Expect**. To test the Nightwatch website, we are going to use both, but we first need to give the specs.

## Specifications

We need a **test suite** with two independent **test cases** that verify the following scenarios:

### Home page

1. Go to http://nightwatchjs.org
2. Wait for the DOM to be ready
3. Check if page title is "Nightwatch.js | Node.js powered End-to-End testing framework"
4. Check if navigation is visible
5. Check if navigation contains a contact link
6. Log all items from navigation
7. Check if the "API Reference" link references the right page
8. Click that link
9. Check if page title is "API Reference | Nightwatch.js"
12. Take a screenshot

### API Reference

1. Go to http://nightwatchjs.org/api
2. Wait for the DOM to be ready
3. Check if the main container has the "secondary" class
4. Check if the main title is "API Reference"
5. Check if there are four items in the sidebar menu
6. Check if the sidebar menu contains "Assert" or "Expect"
7. Take a screenshot
8. Close the test suite

## Nightwatch with Assert (TDD)

The Assert API is an extension of the **Node.js** `assert` module.

```JavaScript
// TDD-style suite with "assert"

var assert = require('assert');

module.exports = {
  // First test case
  'Home': function (browser) {

    function logNav(items) {
      console.log('\nNAVIGATION\n');
      items.value.forEach(function (item) {
        browser.elementIdText(item.ELEMENT, function (res) {
          console.log(res.value);
        });
      });
    }

    browser
      .url('http://nightwatchjs.org/')
      .waitForElementPresent('body', 1000)
      .assert.title('Nightwatch.js | Node.js powered End-to-End testing framework')
      .assert.visible('.navbar ul.nav')
      .assert.containsText('.navbar ul.nav', 'Contact')
      .elements('css selector', '.navbar ul.nav li', logNav)
      .assert.attributeContains('.navbar ul.nav li:nth-of-type(4) a', 'href', '/api')
      .click('.navbar ul.nav li:nth-of-type(4)')
      .assert.title('API Reference | Nightwatch.js')
      .screenshot();
  },

  // Second test case
  'API Reference': function (browser) {

    var sidebarMenu = '#api-container .bs-sidebar > ul > li';

    function testSidebar(items) {
      assert.equal(items.value.length, 4); // Node.js "assert" module
      browser.assert.containsText(sidebarMenu + ':nth-of-type(2)', 'Assert');
    }

    browser
      .url('http://nightwatchjs.org/api')
      .waitForElementPresent('body', 1000)
      .assert.containsText('#api-container h1', 'API Reference')
      .elements('css selector', sidebarMenu, testSidebar)
      .screenshot()
      .end();
  }
};
```

## Nightwatch with Expect (BDD)

The Expect API is an extension of the **Chai** `expect` library.

```JavaScript
// BDD-style suite with "expect"

var expect = require('chai').expect;

module.exports = {
  // First test case
  'Home': function (browser) {

    function logNav(items) {
      console.log('\nNAVIGATION\n');
      items.value.forEach(function (item) {
        browser.elementIdText(item.ELEMENT, function (res) {
          console.log(res.value);
        });
      });
    }

    browser.url('http://nightwatchjs.org/');
    browser.waitForElementPresent('body', 1000);
    browser.expect.element('title').text.to.equal('Nightwatch.js | Node.js powered End-to-End testing framework');
    browser.expect.element('.navbar ul.nav').to.be.visible;
    browser.expect.element('.navbar ul.nav').text.to.contain('Contact');
    browser.elements('css selector', '.navbar ul.nav li', logNav);
    browser.expect.element('.navbar ul.nav li:nth-of-type(4) a').to.have.attribute('href').which.contains('/api');
    browser.click('.navbar ul.nav li:nth-of-type(4)');
    browser.expect.element('title').text.to.equal('API Reference | Nightwatch.js');
    browser.screenshot();
  },

  // Second test case
  'API Reference': function (browser) {

    var sidebarMenu = '#api-container .bs-sidebar > ul > li';

    function testSidebar(items) {
      expect(items.value.length).to.equal(4); // Chai module
      browser.expect.element(sidebarMenu + ':nth-of-type(1)').text.to.contain('Expect');
    }

    browser.url('http://nightwatchjs.org/api');
    browser.waitForElementPresent('body', 1000);
    browser.expect.element('#api-container h1').text.to.contain('API Reference');
    browser.elements('css selector', sidebarMenu, testSidebar);
    browser.screenshot();
    browser.end();
  }
};
```

## Conclusion

Nightwatch.js is a powerful and easy-to-use testing framework. This is a wise choice for your end-to-end tests, even though there are many serious alternatives like [WebdriverIO](http://webdriver.io/), [Protractor](http://www.protractortest.org/) or [DalekJS](http://dalekjs.com/). It is worth a try!
