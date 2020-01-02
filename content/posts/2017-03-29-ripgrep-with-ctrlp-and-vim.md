---
layout: post
title: Using Ripgrep with Vim in CtrlP and Ack.vim
excerpt: "Ripgrep is even faster than Silver Searcher--use with CtrlP and ack.vim for some medium sized performance gains"
tags: [vim, ack, ripgrep, search]
comments: true
date: 2017-03-29
---

Sometimes to improve and broaden my mind I watch the Kardashians. (Who knew such sublime things would one day be televised?)  Anyway last week when Kylie and Kendall 
[nearly came to blows](https://www.youtube.com/watch?v=dQw4w9WgXcQ)
over which plain-text search utility was the fastest, I felt fortunate to have tuned in—"What is ripgrep, and why is Kylie so excited about it,", I wondered aloud.

Well it turns out that ripgrep is the next in a line of improvements to grep, the ole standby text search utility we grew up with.  Turns out ripgrep is fast. As in, maybe 2-5 times as fast as what you are using now.  You can [read this](http://blog.burntsushi.net/ripgrep/) for the details, but suffice it to say that it was worth trying to figure out how to use it within vim for ctrlp (fuzzy file finding) and ack.vim (searching through the current project).  So here is how get that going on:

##### 1. Install ripgrep  (on mac, % brew install ripgrep)

##### 2. Configure ctrlp to use ripgrep (rg) instead of whatever you used to use.  Put this in your .vimrc:

{{< highlight ruby >}}
if executable('rg')
  let g:ctrlp_user_command = 'rg --files %s'
  let g:ctrlp_use_caching = 0
  let g:ctrlp_working_path_mode = 'ra'
  let g:ctrlp_switch_buffer = 'et'
endif
{{< /highlight >}}

##### 3. Configure ack.vim to use ripgrep instead of ag or whatever you used to use.  Put this in your .vimrc:

{{< highlight ruby >}}
  let g:ackprg = 'rg --vimgrep --no-heading'
{{< /highlight >}}

##### 4. Sweet!  You just got a few milliseconds back on every search.  (they do add up)
