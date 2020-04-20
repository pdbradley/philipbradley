+++
title = "Rspec Into Vim With Quickfix"
date = "2015-10-21"
description = "Jump straight to your rspec error locations from Vim using this nifty trick with the quickfix window"
tags = [
    "ruby",
    "rspec",
]
+++

If you live and die by TDD you probably started spending a lot of time writing tests which drive most of the application code you write.  Red, Green, Refactor, etc.  My tool of choice is Rspec, although Minitest is also quite nice.


Maybe I just have lazy eyes, but I get tired of scanning the pane where my tests run to find the exact location of each test failure, then navigating to it.  I'm pretty fast with vim file navigation, but I wanted a better way.


Enter the [quickfix](http://vimdoc.sourceforge.net/htmldoc/quickfix.html)
feature in Vim.  It was originally conceived to save error messages from compilers and allow you to to jump from the list of errors to the actual location of the errors, one by one.  Yeah so that's exactly what we want to do with our Rspec errors, right?  It's just a matter of formatting the rspec output to match what quickfix expects.

So what does quickfix expect?  The vim docs are a little daunting, but one really simple format quickfix understands is:

{{< highlight ruby >}}
<filename_with_relative_path>:<line_number>:<message_about_error>
{{< /highlight >}}

Just three chunks of info separated by two colons.

Furthermore you can open up *any* file as a quickfix file with the vim command 

{{< highlight ruby >}}
:cg <filename>.
{{< /highlight >}}

Just for giggles, go into your rails project, create a file in the root folder called "quickfix" and put this on the first couple lines:

{{< highlight ruby >}}
./Gemfile:5:Just some random place in my project
./config/database.yml:10:I'm on line ten but I'm not sure why!
{{< /highlight >}}

Now open up vim (in that same project) and use the :cg command to open up your quickfix file:

{{< highlight ruby >}}
:cg quickfix
{{< /highlight >}}

You can navigate to the quickfix split and press return on either of those two lines; vim will shoot you to the exact referenced line in each file. 

So what we want is to get our rspec errors to end up in this quickfix format so that they will be in a nice tidy list, and so we can quickly jump straight to the failing test in our code.

Rspec has a few built in formatters, like "documentation" or "progress" that you can specify with the --format flag from the command line.  But you can also define your own [custom formatter](https://www.relishapp.com/rspec/rspec-core/v/3-3/docs/formatters/custom-formatters) with which you override the methods you want to modify.

You just have to make a Ruby class.  I found a few out on the interwebs. There's one at [vim-rspec-quickfix](https://github.com/dapplebeforedawn/vim-rspec-quickfix) but I felt that his formatter was a bit over complicated. Here's a simpler one that I prefer that I found at [digitalronin](http://digitalronin.github.io/coding/2014/10/31/rspec3--vim-quickfix-list/). I modified it ever-so-slightly (name change):

{{< highlight ruby >}}
class QuickfixFormatter
  RSpec::Core::Formatters.register self, :example_failed

  def initialize(output)
    @output = output
  end

  def example_failed(notification)
    @output << format(notification) + "\n"
  end

  private

  def format(notification)
    rtn = "%s: %s" % [notification.example.location, notification.exception.message]
    rtn.gsub("\n", ' ')[0,160]
  end
end
{{< /highlight >}}

What's going on in this class?  You need to **register** the notification that your custom formatter supports (we are only customizing *example_failed* notifications), and then you write a custom method for that type of notification.  Our custom example_failed method adds a notification to @output that consists of the location of the failed example, plus the associated message.

Because I don't want to copy this file into all of my rails projects, I keep it in my ~/code/rspec_support/ folder so all my projects can refer to it.

And you need to tell rspec to use this custom formatter when you run your tests.  Rspec can use multiple formatters at the same time, so you don't have to give up on seeing whatever you normally see when your tests run.  But using your custom formatter will let you generate a quickfix list quietly alongside:

From within my rails projects, now:

{{< highlight ruby >}}
rspec --format progress --require ~/code/rspec_support/quickfix_formatter.rb --format QuickfixFormatter --out quickfix.out spec
{{< /highlight >}}
This runs my specs with "progress" visual formatting, but also requires my custom formatter class and uses it to format the errors into my specified output file, quickfix.out

That's a lot to type so of course it is mapped to my rspec_command in my vimrc like this:

{{< highlight ruby >}}
"Rspec.vim mappings
map <Leader>t :call RunCurrentSpecFile()<CR>
map <Leader>s :call RunNearestSpec()<CR>
map <Leader>l :call RunLastSpec()<CR>
map <Leader>* :call RunAllSpecs()<CR>
let g:rspec_command = 'call VimuxRunCommand("bundle exec spring rspec --format progress --require ~/code/rspec_support/quickfix_formatter.rb --format QuickfixFormatter --out quickfix.out {spec}\n")'
" opens the quickfix file and window
{{< /highlight >}}

And I mapped leader q to open the quickfix window for these errors and focus on it:

{{< highlight ruby >}}
:map <leader>q :cg quickfix.out \| cwindow<CR>
{{< /highlight >}}

Now when I run my tests, I get my usual visual cues in my test pane, but if there are failures, I can quickly pull up a quickfix list in vim with <leader>q to jump to each failure.

Looks kind of like this in real life.  Notice I still get all my rspec detail on the right, but in my vim window I've got the quickfix list at the bottom for easy navigation.

[![Vim Quickfix Rspec][vim-session]][vim-session]

So in closing:

1. Create a little rspec formatter class.  Copy mine from [github](https://github.com/pdbradley/rspec_support) if you want.  Save it somewhere convenient.
2. If you have a shortcut to run your rspec tests (surely you do; if not, why not?) modify it to require the rspec formatter code, use it, and direct its output to a file (quickfix.out is what I use)
3. In your vimrc create a shortcut to open that file as a quickfix file and use it to navigate to your test failures.

If you have a better way to do this, or a better way to format the quickfix output, let me know.  I'm always ready to level up.

[vim-session]: /vimrspecquickfix.png
