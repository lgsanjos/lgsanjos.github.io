---
layout: post
title: "Setup Integration Tests with Capybara"
date: 2016-01-13 22:13:26 -0200
comments: true
categories: [tech, testing]
---

This is a tutorial to setup an entire integration test project based on **selenium**, **capybara** and **phantomjs**. This is a purely black box text, so no matter the language you used on your website it should be helpful.

<!-- more -->

## First things first

### Dependencies
On an empty directory, let's create the Gemfile with a basic content, like:

{% codeblock lang:ruby %}
  source 'http://rubygems.org'

  #specify here your ruby version
  ruby '2.2.2'

  # Use the lastest version first, once it's working fine update this file
  gem 'cucumber'
  gem 'selenium-webdriver'
  gem 'poltergeist'
  gem 'phantomjs', :require => 'phantomjs/poltergeist'
  gem 'rspec'
  gem 'capybara'
{% endcodeblock %}

Save this file and type down on your terminal:

{% codeblock lang:bash %}
  bundle install
{% endcodeblock %}

It is pretty standard, should be fine.

### Setup

Cucumber is the test runner that will look for .feature files and step_definitions. The web thing is all related to selenium or poltegeist, don't bother with this right now, let's stick with the cucumber first.

On the terminal, type down:

{% codeblock lang:bash %}
  cucumber --init
{% endcodeblock %}

it should create a structure like this:

{% codeblock lang:bash %}
.
├── Gemfile
├── Gemfile.lock
├── README.md
└── features
    ├── step_definitions
    └── support
        ├── env.rb
{% endcodeblock %}

That is enough to run cucumber, of course we haven't written any tests so far but it's good to check the engines :). Type "cucumber" on your terminal and you should see this:

{% codeblock lang:bash %}
-> cucumber

0 scenarios
0 steps
0m0.000s
{% endcodeblock %}

## Writing the first scenario

For the first scenario we won't touch the website yet, lets begin with a simple test and go throught the basics of cucumber.

### Creating the test file

The tests should be placed on the feature folder. For the sake of simplicity we will write a test for the login feature (the most vanilla feature I can thing of).

Create a file named "login.feature" on the feature folder and write down a very simple scenario like this:

{% codeblock lang:cucumber %}
Feature: Login

  Scenario: Login failed
    Given I visit my website
    And I'm on the login page
    When I type down a wrong user credentials
    Then a failure modal shows up
    And I should see 'Login failed' as error message
{% endcodeblock %}

This is a tipical example of cucumber scenario, if you run **cucumber** on terminal again, you will see that it found the test but there is no implementation of the step definiitons.

### Step definitions

When you execute cucumber, its engine parses all the ".feature" files and for each step description it tries to match a step definition. As you can see there is already a folder named "Step definitions" where we will place them.

Create a file named **login_steps.rb** with this:

{% codeblock lang:ruby %}
Given /^I visit my website$/ do
end

And /^I'm on the login page$/ do
end

When /^I type down a wrong user credentials$/ do
end

Then /^a failure modal shows up$/ do
end

And /^I should see '(.\*)' as error message$/ do |message|
end
{% endcodeblock %}

As you can see, the definitions are matched by **ruby regex**, before we continue a few considerations:

* The prefix **Given**, **When**, **Then** not necessary need to match. But it would be good if it does :p

* Whatever is inside the bars // is a regex pattern and you can research about ruby regex to understand better.

   * The ^ means the **string should start** with that pattern.

   * The $ means the **string should end** with that pattern.
 
   * The . means **any character**.

   * The * means **any amount of repetitions**.

Now your tests should run successfully, because we have no implementations yet.

## Configuring Capybara

### Env.rb

This file will load before the tests execution, you should place here all the environment realted setup. Don't think of this as a hook to clean database or anything like that, later I'll show you a better place to do it.

{% codeblock lang:ruby %}
require 'capybara/cucumber'
# this is will be important so rspec will use capybara's matchers
include Capybara::Node::Matchers

  driver = :selenium
  Capybara.configure do |config|
      config.app_host = 'http://yourdomain.com/'
      config.run_server = false
      config.default_driver = driver
      config.current_driver = driver
      config.javascript_driver = driver
  end
{% endcodeblock %}

This configuration is enough to setup capybara with the selenium webdriver and notice the variable **run_server=false** it means, your website should be running before you run the tests.

PS: The files inside support folder are standard ruby files, so you can add "puts", read system variables, create function or simply require other files to better organize your settings.

## The driver

### What it is?

There are several complexities to communicate with the browser, each browser expose a different API, each platform has it's difference as well, so there are several abstractions to aliviate the developer of all this pain. The driver is one this abstraction on the browser level, so what you write in selenium running on firefox should run fine on chrome.

Capybara is another level of abstraction to handle multiple drivers also provides a DSL that is ruby-like and makes our life much easier.

### Your best friend: page

Back to the test, each test scenario should to open the browser, load your URL and then comunicate with the loaded page. To do it we will use a global variable named **page**, it represents all the DOM and expose methods that you can interact with it.

Back on our step definitions, on the method 'I visit my Website' add this (make sure to set your url).

{% codeblock lang:ruby %}
    visit 'http://localhost:3000/'
{% endcodeblock %}

This command will open the browser and navigate to the specified URL.

{% codeblock lang:ruby %}
   expect(page).to have_title 'My Title'
{% endcodeblock %}

   And here we are using the **rspec** matchers to help us assert the page title. The variable **page** is assigned after **visit** is called, so you must call first **visit**.

**Tip:** I like to add this validation just after visiting a website, it will force the test to wait until the page is loaded so it will move on to the next step.

### The API

I won't get in details here because there is a good documentation for Capybara and lot's of examples all over the internet, take a look on the following links and you should be fine.

 * [Capybara DOCS](http://www.rubydoc.info/github/jnicklas/capybara/master#navigating)
 * [Capybara examples](https://github.com/jnicklas/capybara)

### Example

One possible way to implement all the steps is like this:

{% codeblock lang:ruby %}
Given(/^I visit my webpage$/) do
    visit 'hhtp://localhost:3000/'
end

And(/^Iim on the login page$/) do
    expect(page).to have_title 'Login'
end

When(/^I type down a wrong user credentials$/) do
    fill_in :login, :with => 'wrong login'
    fill_in :password, :with => 'wrong password'
    click_button 'Sign in'
end

Then(/^a failure modal shows up$/) do
    expect(page).to have_selector '.modal-dialog'
end

And(/^I should see '(.\*)' as error message$/) do |message|
    page.has_selector?('div#error_message', :text => message)
end
{% endcodeblock %}

## Bonus

### Hooks

To execute code **before** or **after** each scenario/test you can use hooks, by creating a file named hooks on the support folder, with the given content:

{% codeblock lang:ruby %}
Before do
    # runs before each scenario
end

Before do |scenario|
  # Same as previous but receive the scenario as argument just in case you want to get the title or description.
  Rails.logger.debug "Starting scenario: #{scenario.title}"
end

After do
    # runs after each scenario
    # For all hooks the scenario argument is optional, so it would work After do |scenario|  as well
end

BeforeStep
  # Do something before each step.
end

AfterStep
  # Do something after each step.
end

Around('@fast') do |scenario, block|
    block.call
    #It select all scenarios tagged as @fast and will execute the test when you execute block.call
end
{% endcodeblock %}

[Official documentation](https://github.com/cucumber/cucumber/wiki/Hooks)

### PhantomJS

Tired of the browser poping up on your screen? Do you want to run it on a server with no UI? Headless browser are the solution, a very handy one is PhantomJS and it's driver Potergeist :P

To install PhantomJs follow this link [http://phantomjs.org/](http://phantomjs.org/)

Once it is installed, the setup we did above is enough to use it! Just change the **env.rb** file, assign the driver variable to **:poltergesit**.

It will run all the tests against a headless browser, it's much faster but you may have inconsistent results because it's a different browser.

### Page Objects

As you write more and more tests, you will notice that there are duplications and the maintance becomes expensive, for instance if the page title has changed, now you need to update all the steps that assert the page title.

**Page Objects** is a concept that everything related to a page whould be encapsulated into an object and the steps definition would interact only with this object.

Based on the example we did here, the implementation could be like this:

{% codeblock lang:ruby %}
Given(/^I visit my webpage$/) do
   @loginPage = LoginPage.visit
end

And(/^Iim on the login page$/) do
  expect(@loginPage.displayingCorrectTitle).to be_truthy
end

When(/^I type down a wrong user credentials$/) do
    @loginPage.login 'wrong login'
    @loginPage.password 'wrong password'
    @loginPage.sign_in
end

Then(/^a failure modal shows up$/) do
    expect(@loginPage.showingErrorDialog?).to be_truthy
end

And(/^I should see '(.\*)' as error message$/) do |message|
    expect(@loginPage.displayingCorrectErrorMessage?).to be_truthy
end
{% endcodeblock %}

Don't take the nomeclature as guideline, but the idea of extracting all the interactions to a centralized object to each page you have. So updating this object will impact all the scenarios.

More about page objects here:

 * [Page Object Concept](http://martinfowler.com/bliki/PageObject.html)
 * [A gem to help you](https://github.com/cheezy/page-object)

