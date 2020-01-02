---
layout: post
title: Using Capybara to Tee Up an Interactive Browser Session Quickly
excerpt: "Capybara can automate some of the repetitive setup for actual click and type testing that you want to do"
tags: [ruby, capybara, rails, rspec]
comments: true
date: 2016-03-30
---

Even though the stuff I make generally is test driven, there always comes a time when you need to click around your web app like a real person would.

In development, I often use a rake task to populate the database with sample data to get a sense of how things will look and feel with "real" data.  But I really loathe needing to log in and navigate to various pages that I'm inspecting.  Especially when I need to do it again and again.

[They say the best programmers are the lazy ones](http://threevirtues.com), and if that's true I have a bright future, because using [Capybara](https://github.com/jnicklas/capybara) I find a pretty easy way to automate getting to the place I want to be in my web app.

It isn't hard to get Capybara to navigate through a web path; the hard part is keeping Capybara from killing the browser when it finishes.  But really all you have to do is override the "quit" method on the driver class, like this:

{{< highlight ruby >}}

session = Capybara::Session.new(:selenium)
session.driver.class.class_eval { def quit; end }

{{< /highlight >}}

I got this little tip from a [strangely undervoted answer](http://stackoverflow.com/questions/7555416/how-to-leave-browser-opened-even-after-selenium-ruby-script-finishes) by Eliot Larson on Stackoverflow.

Now if you run a script that uses Capybara, it will fire up a browser, do what you tell it, and then exit, but the browser window stays open for you to take over.

I use the script below to log into one application as either a manager, a tutor, or a client, and I make some assumptions about the data in the app (usernames and passwords) but I've also used this approach to actually create a new account each time.

Here is the whole file, which I store in script/login, and use by typing % script/login tutor or % script/login manager etc.

{{< highlight ruby >}}

#!/usr/bin/env ruby
require 'capybara'

role = ARGV[0]
exit unless %w(client manager tutor).include? role

session = Capybara::Session.new(:selenium)
session.driver.class.class_eval { def quit; end }
session.visit('http://127.0.0.1:3000')

case role
when 'tutor'
  session.click_link "Sign in as a Tutor"
  session.fill_in "Email", with: "tutor1@example.com"
  session.fill_in "Password", with: "123123"
when 'manager'
  session.click_link "Sign in as a Manager"
  session.fill_in "Email", with: "manager1@example.com"
  session.fill_in "Password", with: "123123"
when 'client'
  session.click_link "Sign in as a Client"
  session.fill_in "Email", with: "client1@example.com"
  session.fill_in "Password", with: "123123"
end

session.click_button "Log in"

{{< /highlight >}}
