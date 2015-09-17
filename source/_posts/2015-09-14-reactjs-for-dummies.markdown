---
layout: post
title: "ReactJS for dummies"
date: 2015-09-14 11:24:21 -0300
comments: true
categories: tech
---

After almost two years working with AngularJS a new project came to me with few presentation rules but high performance was requires. So I took a look again on the frameworks around and decided to give a try for ReactJS. Here are my thoughts about it so far, and some quick explanation for those who never saw it before.

<!-- more -->

## A brief intro

ReactJs is a tool developed by Facebook to generate and manipulate DOM (Document Object Model, it is a tree of objects that the browser produces when parse the HTML). You can think of ReactJS as the V on MVC, it does not provide you service layer nor controllers, nor routing by default. Sure there are plugins and ways o implement this but it's beyond the standard.

For now let's keep it simple and think on view only. React is very efficient mostly because  it keeps in memory it's own version of the DOM tree, it means that it reduces a lot the comunication between browser which apparently is expensive.

## What else?

It drives you to encapsulate **HTML** and Javascript in components, let's say you have a table and a header that allow the user to reorder the collumns, in general manner you would select the table **id** and change the html order, without any regard of boundaries, you may implement the javascript in a separate file with a completely different name than the HTML. So we treat the whole HTML, CSS as global, right?

On ReactJS way, you would encapsulate the JS that manipulates that specific HTML in one component.

## But how does that look like? 

The simplest example:

{% codeblock lang:js %}
  var MyComponent = React.createClass({

    // This is an API method that must be implemented and return the html that will be rendered
    render: function () {
      return (<div>Hello world</div>);
    }
    
 }); 
{% endcodeblock %}

## What we can notice

+ Now we are using JS to generate html
+ A piece of html is being encapsulated inside a JS component
+ A piece of html is being returned as first class type
+ There are methods that API relies and are not obvious for me

## Explanation

First of all, this is not pure JS it is actually called JSX, so it should be precompiled to generate proper "browser friendly input". How to precompile?
Well, you can use **NodeJS** as engine and a task executer like **grunt** or **gulp** that will trigger as part of deploy/test/preview task to apply the ReactJS precompiler.

This explain how you can return HTML, which in my opinion is great, so you can break down your render function calling other JS functions that also return HTML.

Sure there are also a few downsides, for instance the attribute **class** that we use to set a **CSS** class should be named as **classFor**, the reason is... well it's the way it is to work.

Some more examples:

## Using JS to populate the content

{% codeblock lang:js %}
  var MyComponent = React.createClass({
  
  var toBeDisplayed = [ 'Hello', 'world', 'my', 'friend'],

  print: function (list) {
    list.map(function (word) {
      return (<p>{word}</p>);
    });
  },

  render: function () {
    return (<div>{this.print(toBeDisplayed)}</div>);
  }
    
  }); 
{% endcodeblock %}

## Hold on!

It's **not recommended** to use variables inside component because components have the concept of state, and when a state change it assumes that the UI needs to be rerendered.
Take a look below:

## Changing the state and update the UI

{% codeblock lang:js %}
  var MyComponent = React.createClass({
 
  // returns an object which contain all the state variable with default value 
  getInitialState: function() {
   return {
       toBeDisplayed: 'Default value';
     };
  },

  updateDataAsync: function () {
    setTimeout(function () {
      this.state.toBeDisplayed = 'I changed this data!';
      
    }, 2000);
  },

  // This is also from API, it triggers once the render is done
  componentDidMount: function () {
    this.updateDataAsync();
  },

  render: function () {
    return (<div>{this.state.toBeDisplayed}</div>);
  }
    
  }); 
{% endcodeblock %}


## Components composition and parameters

{% codeblock lang:js %}
  
  var FancyPrint = Ract.createClass({
    render: function () {
      return (<h1>this.props.text</h1>);  
    }
  });

  var MyComponent = React.createClass({

    render: function () {
      return (<FancyPrint text='Hello world' />);
    }
    
 }); 
{% endcodeblock %}

## this.props ?

Yes, whatever you get as arguments are properties and you should handle them as constants inside your component, if you need to manipulate and change you better move the value to the State.

Also you can pass an event as property for sure.


## Overall
- Component driven feels like building the HTML in a **OO** fashion.
- Based on my experience I can tell you that there are very few situations which the standard attributes were changed and usually is easy to find references on internet.
- One thing that bothers me is when special methods are triggered and is not clear to me the whole lifecycle, for React there are few events I'll show later.

## Ok so far? Let's imagine... 

I won't write down the entire code here, it's not my intention to write a tutorial right now, but probably it will be my next post.

But what do you think, if your main page could look like this:

{% codeblock lang:js %}
var MetadataHeader = require('scripts/MetadatHeader.js');
var PageHeader = require('scripts/PageHeader.js');
var SideMenu = require('scripts/SideMenu.js');
var MainContent = require('scripts/MainContent.js');
var Footer = require('scripts/Footer.js');

var Page = React.createClass({

  getInitialState: function () {
    return { currentPage: 'index' };
  },
  
  changeContent: function (next) {
    this.state.currentPage = next;
  },

  render: function () {
    return (
      <html>
        <MetadataHeader />
        <body>
          <PageHeader />
          <SideMenu onClickMenu={this.changeContent}/>
          <MainContent display={this.state.currentPage}/>
          <Footer />
        </body>
      </html>
    );
  }
});
{% endcodeblock %}

Doesn't look great? The best part is that any javascript that updates the DOM, MUST be placed inside the componente that renders the DOM. So it keeps organized and React will render only whatever is necessary.

Parent component can pass argumets to its children, arguments can be variables or a callback event. So a children component can notify other components when something happen.

## Extras points

- **Official website:** <http://facebook.github.io/react/>
- **Community:** There are a good amount of free plugins available, take a look: <http://react.rocks/>
- **Cheatsheet:** A good reference <http://ricostacruz.com/cheatsheets/react.html>
- **Yeoman and generators** This is a easy simple way to setup a new project, this generator will create gulp and it's test/preview/deploy pipeline for free <https://github.com/randylien/generator-react-gulp-browserify>
