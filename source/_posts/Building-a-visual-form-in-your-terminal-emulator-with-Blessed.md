---
title: Building a visual form in your terminal emulator with Blessed
date: 2017-06-05 23:52:43
categories:
  - JavaScript
tags:
  - Blessed
  - Node.js
  - ES5
  - Terminal
thumbnail: /images/javascript.png
---

Do you like **ncurses**? Do you like **JavaScript**? If yes, you will love [**Blessed**](https://github.com/chjj/blessed); a Node.js library to create highly interactive and rich text-based user interfaces.

<!-- more -->

## GUI vs CLI vs TUI

**Graphical User Interfaces** (GUIs) are great in terms of design and usability, but they are quite poor when it comes to automation and performance. For complex operations, it is much easier to write a shell script with Bash than programming macros to interact with graphical components. Of course, sometimes we have to test GUIs. But do we need to automate heavy ones when we just want to manipulate files and perform basic operations? The answer is obviously NO.

However, a **Command Line Interface** (CLI) is not very user-friendly. For this reason, we might need a better interface, without sacrificing too much performance. Here comes the **Text-based User Interface** (TUI)...

The principle is simple: we are still in a terminal, we are still in a textual environment, but here we can create visual layouts using widgets. Graphical components are lightweight and can be used to reproduce layouts we can find on most desktop or web applications.

## Specifications

Let's say we want to get some information about a user. We need to know:

- The full name (first name + last name) of the user
- His favorite text editors among "Vim", "Emacs", "Atom" and "Brackets"
- If he likes Blessed or not
- If he has some comments to share

## Basic CLI implementation with Bash

To get user input, we can create a Bash script and use the `read` command. A possible (but very basic) implementation could look like this:

```Bash
#!/bin/bash

echo -e "\033[0;34mFirst name:\033[0m"
read -r firstname

echo -e "\033[0;34mLast name:\033[0m"
read -r lastname

echo -e "\033[0;34mWhat are your favorite editors? (enter numbers only)
 1) Vim
 2) Emacs
 3) Atom
 4) Brackets\033[0m"
read -r selection

editors=""
if [[ "$selection" =~ ^.*1.*$ ]]; then editors+="Vim "; fi
if [[ "$selection" =~ ^.*2.*$ ]]; then editors+="Emacs "; fi
if [[ "$selection" =~ ^.*3.*$ ]]; then editors+="Atom "; fi
if [[ "$selection" =~ ^.*4.*$ ]]; then editors+="Brackets "; fi

echo -e "\033[0;34mDo you like Blessed? (y/n)\033[0m"
read -r blessed

echo -e "\033[0;34mYour comments...\033[0m"
read -r comments

echo "$firstname $lastname
--------------------------
Favorite editors: $editors
Likes Blessed: $blessed
Comments: $comments" > data.txt
```

If I run the script and provide all relevant information, a `data.txt` file is generated with the following content:

```
Baptiste Vannesson
--------------------------
Favorite editors: Vim Atom 
Likes Blessed: y
Comments: It works!
```

This solution is acceptable if we do not need a sophisticated interface. But to shine in society, we may need something more advanced... Blessed comes to the rescue!

## Advanced TUI implementation with Blessed

Here is the layout we are going to build programmatically:

![Blessed form](/Building-a-visual-form-in-your-terminal-emulator-with-Blessed/blessed.png)

### Importing modules

Here we use the **CommonJS** syntax. We need two modules only: **blessed** and **fs**.

```JavaScript
var blessed = require('blessed'),
    fs = require('fs');
```

### Creating a screen object

Just like the scene in [three.js](https://threejs.org/), we need a primary object to render something on screen. With Blessed, this is easy to memorize because it is actually the **screen** object (which has a `render()` method). The following code creates a screen object called "Blessed form" with smart CSR (change-scroll-region) for a more efficient rendering:

```JavaScript
var screen = blessed.screen({
  smartCSR: true,
  title: 'Blessed form'
});
```

Because we are building a TUI, we can set global key bindings to interact more easily with our interface. A must-have is a shortcut that allows us to quit instead of killing the current shell session. By convention, we will press <kbd>Q</kbd> to quit:

```JavaScript
screen.key('q', function () {
  this.destroy();
});
```

### Configuring widgets

Creating user interfaces with Blessed is not very hard, but extremely verbose and declarative. The library makes an intensive use of the **object specifier** pattern (aka **options object**).

> It sometimes happens that a constructor is given a very large number of parameters. This can be troublesome because it can be very difficult to remember the order of the arguments. In such cases, it can be much friendlier if we write the constructor to accept a single object specifier instead. That object contains the specification of the object to be constructed.

> **Douglas Crockford**, *JavaScript: The Good Parts*

In fact, Blessed is based on **widgets** (text inputs, checkboxes, radio buttons, etc.) that can be configured quite precisely. As an example, here we create the form and its first text box:

```JavaScript
var form = blessed.form({
  parent: screen,
  width: '90%',
  left: 'center',
  keys: true,
  vi: true
});

var label = blessed.text({
  parent: screen,
  top: 3,
  left: 5,
  content: 'FIRST NAME:'
});

var firstName = blessed.textbox({
  parent: form,
  name: 'firstname',
  top: 4,
  left: 5,
  height: 3,
  inputOnFocus: true,
  content: 'first',
  border: {
    type: 'line'
  },
  focus: {
    fg: 'blue'
  }
});
```

Blessed positioning is a bit less convenient and powerful than CSS positioning, but we do not have the same constraints for a web page and a terminal app. For a TUI, absolute positioning does the job quite well...

### Handling events

To submit or reset the form, we have two main buttons: `submit` & `reset`. These buttons must trigger the appropriate form action. It could be done like so:

```JavaScript
submit.on('press', function () {
  form.submit();
});

reset.on('press', function () {
  form.reset();
});
```

When the form is submitted, we have to parse its data properly and write the result to a file. When it is reset, we just send a message (a toast):

```JavaScript
form.on('submit', function (data) {
  // Checkboxes return an array of booleans like [true, false, true, false]
  var editors = ['Vim', 'Emacs', 'Atom', 'Brackets'].filter(function (item, index) {
    return data.editors[index];
  });

  msg.display('Form submitted!', function () {
    var summary = '';
    summary += data.firstname + ' ' + data.lastname + '\n';
    summary += '------------------------------\n';
    summary += 'Favorite editors: ' + editors + '\n';
    summary += 'Likes Blessed: ' + data.like[0] + '\n';
    summary += 'Comments: ' + data.comments;

    fs.writeFile('form-data.txt', summary, function (err) {
      if (err) throw err;
    });
  });
});

form.on('reset', function () {
  msg.display('Form cleared!', function () {});
});
```

Done!

### Full implementation

If your environment is properly set up, you can run the following script with `node`:

```JavaScript
var blessed = require('blessed'),
    fs = require('fs');

// Screen

var screen = blessed.screen({
  smartCSR: true,
  title: 'Blessed form'
});

// Form

var form = blessed.form({
  parent: screen,
  width: '90%',
  left: 'center',
  keys: true,
  vi: true
});

// Text boxes

var label1 = blessed.text({
  parent: screen,
  top: 3,
  left: 5,
  content: 'FIRST NAME:'
});

var firstName = blessed.textbox({
  parent: form,
  name: 'firstname',
  top: 4,
  left: 5,
  height: 3,
  inputOnFocus: true,
  content: 'first',
  border: {
    type: 'line'
  },
  focus: {
    fg: 'blue'
  }
});

var label2 = blessed.text({
  parent: screen,
  content: 'LAST NAME:',
  top: 8,
  left: 5
});

var lastName = blessed.textbox({
  parent: form,
  name: 'lastname',
  top: 9,
  left: 5,
  height: 3,
  inputOnFocus: true,
  content: 'last',
  border: {
    type: 'line'
  },
  focus: {
    fg: 'blue'
  }
});

// Check boxes

var label3 = blessed.text({
  parent: screen,
  content: 'What are your favorite editors?',
  top: 14,
  left: 5
});

var vim = blessed.checkbox({
  parent: form,
  name: 'editors',
  content: 'Vim',
  top: 16,
  left: 5
});

var emacs = blessed.checkbox({
  parent: form,
  name: 'editors',
  content: 'Emacs',
  top: 16,
  left: 20
});

var atom = blessed.checkbox({
  parent: form,
  name: 'editors',
  content: 'Atom',
  top: 16,
  left: 35
});

var brackets = blessed.checkbox({
  parent: form,
  name: 'editors',
  content: 'Brackets',
  top: 16,
  left: 50
});

// Radio buttons

var label4 = blessed.text({
  parent: screen,
  content: 'Do you like Blessed?',
  top: 19,
  left: 5
});

var radioset = blessed.radioset({
  parent: form,
  width: '100%',
  height: 5,
  top: 21
});

var yes = blessed.radiobutton({
  parent: radioset,
  name: 'like',
  content: 'Yes',
  left: 5
});

var no = blessed.radiobutton({
  parent: radioset,
  name: 'like',
  content: 'No',
  left: 15
});

// Text area

var label5 = blessed.text({
  parent: screen,
  content: 'Your comments...',
  top: 24,
  left: 5
});

var textarea = blessed.textarea({
  parent: form,
  name: 'comments',
  top: 26,
  left: 5,
  height: 7,
  inputOnFocus: true,
  border: {
    type: 'line'
  }
});

// Submit/Cancel buttons

var submit = blessed.button({
  parent: form,
  name: 'submit',
  content: 'Submit',
  top: 35,
  left: 5,
  shrink: true,
  padding: {
    top: 1,
    right: 2,
    bottom: 1,
    left: 2
  },
  style: {
    bold: true,
    fg: 'white',
    bg: 'green',
    focus: {
      inverse: true
    }
  }
});

var reset = blessed.button({
  parent: form,
  name: 'reset',
  content: 'Reset',
  top: 35,
  left: 15,
  shrink: true,
  padding: {
    top: 1,
    right: 2,
    bottom: 1,
    left: 2
  },
  style: {
    bold: true,
    fg: 'white',
    bg: 'red',
    focus: {
      inverse: true
    }
  }
});

// Info

var msg = blessed.message({
  parent: screen,
  top: 40,
  left: 5,
  style: {
    italic: true,
    fg: 'green'
  }
});

var table = blessed.table({
  parent: screen,
  content: '',
  top: 40,
  left: 'center',
  style: {
    fg: 'green',
    header: {
      bold: true,
      fg: 'white',
      bg: 'blue'
    }
  },
  hidden: true
});

// Event management

submit.on('press', function () {
  form.submit();
});

reset.on('press', function () {
  form.reset();
});

form.on('submit', function (data) {
  var editors = ['Vim', 'Emacs', 'Atom', 'Brackets'].filter(function (item, index) {
    return data.editors[index];
  });

  msg.display('Form submitted!', function () {
    var summary = '';
    summary += data.firstname + ' ' + data.lastname + '\n';
    summary += '------------------------------\n';
    summary += 'Favorite editors: ' + editors + '\n';
    summary += 'Likes Blessed: ' + data.like[0] + '\n';
    summary += 'Comments: ' + data.comments;

    fs.writeFile('form-data.txt', summary, function (err) {
      if (err) throw err;
    });
  });
});

form.on('reset', function () {
  msg.display('Form cleared!', function () {});
});

// Key bindings

screen.key('q', function () {
  this.destroy();
});

// Render everything!

screen.render();
```

## Conclusion

Building an advanced TUI may be a long process. Sometimes a raw CLI program is better, but if usability really matters, Blessed can be an interesting option and a good alternative to ncurses.

By the way, in the use case presented here, there is no validation at all: not in our Bash script, nor in our Blessed script. If you want a good exercise, maybe you can try to implement it...
