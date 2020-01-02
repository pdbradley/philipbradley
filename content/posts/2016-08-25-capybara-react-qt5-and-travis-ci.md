---
layout: post
title: Installing QT5 on Travis-CI to use Capybara Webkit with React Components
excerpt: "Here is what you need to put in your travis.yml file to be able to test react components using capybara-webkit"
tags: [ruby, capybara, rails, rspec]
comments: true
date: 2016-08-25
---

Because this took me so long to figure out, I'm posting it.  On some capybara feature tests, my react components would not render (on Travis).

Eventually I figured out that this was because I was missing qt5 on Travis (Ubuntu 12.04)

Anyhow, after about 3 HOURS of tinkering, this is what works.  The main thing to remember is you have to put dist: trusty in your config.

{{< highlight ruby >}}

# put this in your .travis.yml near the top
dist: trusty

# and this in the before_install options.
# The QMAKE line is required to get capybara-webkit to work with qt5
before_install:
- sudo add-apt-repository --yes ppa:ubuntu-sdk-team/ppa
- sudo apt-get update -qq
- sudo apt-get install -qq libqt5webkit5-dev qtdeclarative5-dev
- export QMAKE=/usr/lib/x86_64-linux-gnu/qt5/bin/qmake

{{< /highlight >}}
