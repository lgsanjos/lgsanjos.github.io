---
layout: post
title: "Using BASH and Git to track your environment growth"
date: 2015-09-21 17:21:58 -0300
comments: true
categories: tech
---

The other day I noticed that my test suite was taking over 30 seconds to run. It bothered me and got me thinking **"It wasn't like this..."** so I decided to dig in and I realized that there were several small issues that were slowing it down. The fix was straight foward and in the end it was running in 10 seconds (yay!), but I was still concerned with the question: **"How it got this bad and no one have noticed?"**

This is **related to peripheral** view, slightly changes on our environment doesn't come to our attention. Mostly because it's not our main goal neither we track them, so it changes we don't notice until it's that bad. 

The **solution** is to keep track of data and manage its growth.

<!-- more -->

## Credits

The script bellow is based on a [destroy all software](http://www.destroyallsoftware.com/) podcast that I saw a while ago.

## Step 1 - Benchmark

We know that execution time that our test framework shows is not so reliable, cause it may or may not count the loading time. To be clear on loading time, if you **rake test** on an empty rails project it takes a few seconds to load all the rake, ruby and rails environment even if there is no test to run. This is the loading time.

Te benchmark properly I recommend to use the command **time**.

Try this on your bash:

{% codeblock lang:bash %}
time ls
{% endcodeblock %}

**or** try with your test tool

{% codeblock lang:bash %}
time grunt test
{% endcodeblock %}

It gives you the real execution time of a given process also it's an agnostic way to benchmark test and don't rely on your framework output.

## Now what?

Once we have a good way to benchmark our test execution, now we need to run this for on old revisions of our code.

## Step 2 - Running tests on each revision

Here is pretty standart, we use git rev-list to display all the hashes from commit 0 to the HEAD.

{% codeblock lang:bash %}
git rev-list HEAD --reverse
{% endcodeblock %}

Use the reverse order because we want from oldest to the newest.

Before moving on let's restrict the amount of revisions we will use. Just to save time and so you can run it every once in a while.

{% codeblock lang:bash %}
git rev-list HEAD...HEAD~3 --reverse
{% endcodeblock %}

So now it should list 3 revisions only.
And now with this list of commits, we can parse and use each revision using **while read**

{% codeblock lang:bash %}
 while read *name_a_variable* ; do *insert your code here* ; done
{% endcodeblock %}

so applying it...

{% codeblock lang:bash %}
git rev-list HEAD...HEAD~3 --reverse | while read rev; do git log $rev -1 --oneline; done
{% endcodeblock %}

This will be enough to test the while statement, it will pick each revision and display the log for it. I know it's useless but it is good to test each improvement isolated.

## Time to recap

Fair enough, lets recap: Now we are iterating over each revision on the git history and we are able to run some code on each revision. So now we need to revert our code to that revision, run the tests and capture the running time. To do so you can use **git checkout $rev**. 

{% codeblock lang:bash %}
git rev-list HEAD...HEAD~3 --reverse | while read rev; do git log $rev -1 --oneline && git checkout -q $rev && grunt test; done
{% endcodeblock %}

**What happened here:**

 * For each revision it will print the commit message
 * Do a checkout -q which silently will go back in time and show the old code
 * Run the tests, for me "grunt test" is what I use
 * PS: I like to use && to separate the statements so If one fails it doesn't go on

As we saw on the beggining let's use the command **time** to capture a more accurate execution time. But before running I will give you a tip.

Try to **grep** the time output:

{% codeblock lang:bash %}
time git status | grep real
{% endcodeblock %}

It will show something like this:

{% codeblock lang:bash %}
real  0m0.448s
user  0m0.026s
sys 0m0.310s
{% endcodeblock %}

It happens because time uses the unix error pipe to print the time, so a simple way is to redirect the error pipe to the success pipe and grep it:

{% codeblock lang:bash %}
(time git status) 2>&1 | grep real
real  0m0.448s
{% endcodeblock %}

Now we can apply this to the script:

{% codeblock lang:bash %}
git rev-list HEAD...HEAD~3 --reverse | while read rev; do git log $rev -1 --oneline && git checkout -q $rev && (time grunt test) 2>&1 | grep real ; done
{% endcodeblock %}

The output still is not the best, for me I see this:

{% codeblock lang:bash %}
6ca82b199faf7453e65543e1015896792d3a76ab Adding instructions on how to run sidekiq on README
real  0m2.204s
0003437a0775b1c6df6a6306f68e1c1dcd567400 Solving minor sonar issues
real  0m1.902s
0003437a0775b1c6df6a6306f68e1c1dcd567400 Removed dead code
real  0m2.560s
{% endcodeblock %}

# Step 3 - Collect the time output

Right now we have all the information on screen it is a matter of parsing and collecting them. We can create a table-like output. Just remove this entire line with commit message and use only the revision hash and the time consumed:

{% codeblock lang:bash %}
git rev-list HEAD...HEAD~3 --reverse | while read rev; do git checkout -q $rev && echo "$rev `(time grunt test 2>&1) | grep real`"; done
{% endcodeblock %}

The output will be like:

{% codeblock lang:bash %}
6ca82b199faf7453e65543e1015896792d3a76ab real 0m2.036s
e68fce931a1adeb31a639a013b93b7b301ae96c7 real 0m1.865s
0003437a0775b1c6df6a6306f68e1c1dcd567400 real 0m2.663s
{% endcodeblock %}

So it's a matter of removing this real, we can do using awk:

{% codeblock lang:bash %}
git rev-list HEAD...HEAD~3 --reverse | while read rev; do git checkout -q $rev && echo "$rev `(time grunt test) 2>&1 | grep real`" | awk '{print $1 " " $3}'; done
{% endcodeblock %}

{% codeblock lang:bash %}
6ca82b199faf7453e65543e1015896792d3a76ab 0m2.191s
e68fce931a1adeb31a639a013b93b7b301ae96c7 0m2.049s
0003437a0775b1c6df6a6306f68e1c1dcd567400 0m2.487s
{% endcodeblock %}

Now it's good enough to collect the data, if you want date of the commit it's a matter of retrieve this info using the $rev hash and add inside the **echo**.

## Step 4 - Almost done! Run to a sample of history

According to the ammount of commits you have, you may not want to run for all commits but for a sample of the history, to do so AWK allow us to pick lines that satisfy a **MOD** codition:

{% codeblock lang:bash %}
git rev-list | awk 'NR % 100 == 0'
{% endcodeblock %}

NR is the register number or line number if you will. To better estimate the size of your sample check the ammount of commits you have:

{% codeblock lang:bash %}
git rev-list | wc -l
{% endcodeblock %}

**wc -l** is a line counter, so it works very well with a list like this.

Adding this sample restriction back to the script:

{% codeblock lang:bash %}
git rev-list HEAD --reverse | awk 'NR % 50 == 0' | while read rev; do git checkout -q $rev && echo "$rev `(time grunt test) 2>&1 | grep real`" | awk '{print $1 " " $3}'; done
{% endcodeblock %}

With this you can draw a chart of the time consuming evolution over time. Also if you notice that from one commit to another there is a big gap, dive in and investigate what changed on that rage of commits. 

It's nice to radiate this with your team so they will be aware.

There are other metrics that can be extracted, like the size of fizes, files mostly touched and other crazy metrics your imagination let you think of.

I hope you find this useful ;)
