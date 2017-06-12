---
title: 'End-to-End testing with Nightwatch.js: TDD vs BDD'
categories:
- JavaScript
tags:
- Nightwatch.js
- Testing
- Automation
- TDD
- BDD
---

```JavaScript
// TDD-style suite with "assert"

var assert = require('assert');

module.exports = {
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
      .assert.containsText('#api-container h1', 'API Reference');
  },

  'API Reference': function (browser) {

    var sidebarMenu = '#api-container .bs-sidebar > ul > li';

    function testSidebar(items) {
      assert.equal(items.value.length, 4);
      browser.assert.containsText(sidebarMenu + ':nth-of-type(2)', 'Assert');
    }

    // API Reference
    browser
      .url('http://nightwatchjs.org/api')
      .waitForElementPresent('body', 1000)
      .assert.containsText('#api-container h1', 'API Reference')
      .elements('css selector', sidebarMenu, testSidebar);

    // End
    browser
      .screenshot()
      .end();
  }
};
```

```JavaScript
// BDD-style suite with "expect"

var expect = require('chai').expect;

module.exports = {
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
    browser.expect.element('#api-container h1').text.to.contain('API Reference');
  },

  'API Reference': function (browser) {

    var sidebarMenu = '#api-container .bs-sidebar > ul > li';

    function testSidebar(items) {
      expect(items.value.length).to.equal(4);
      browser.expect.element(sidebarMenu + ':nth-of-type(1)').text.to.contain('Expect');
    }

    // API Reference
    browser.url('http://nightwatchjs.org/api');
    browser.waitForElementPresent('body', 1000);
    browser.expect.element('#api-container h1').text.to.contain('API Reference');
    browser.elements('css selector', sidebarMenu, testSidebar);

    // End
    browser
      .screenshot()
      .end();
  }
};
```
