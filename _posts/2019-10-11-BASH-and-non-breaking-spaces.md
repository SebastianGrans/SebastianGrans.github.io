---
layout: post
title: BASH and non-breaking spaces
---

<center>
<img src="{{ site.baseurl }}/images/20191011-NBSP/headerimg.png" alt="Non-breaking space causes headaches" style="display: block;"/>
</center>


<div markdown="1" style="display:block; background-color:#eff0f1; border-radius: 3px; padding: 0px 8px;">
**TL;DR:** BASH says that "file name" ≠ "file name" because the latter has a non-breaking space. For more information see [this](https://www.thelinuxrain.com/articles/how-to-deal-with-nbsps-in-a-terminal) link.
</div>

I sometimes uses spaces in my filenames because I think it looks better. Most often it never causes a problem, but then every once in a while it comes back and bites me in the ass.

In this case I had manually created a file called `04 Monday.md`, and without noticing I had pressed ⎇+space (alt+space) while writing the filename. On a Mac this inserts a _non-breaking space_ rather than a regular space.

Later I needed to do some BASH scripting involving this file and  `[ -f "04 Monday.md" ]` returned `false`. After a lot of head scratching I also noticed that `ls 04\ Monday.md` didn't work either and then it dawned on me that I had had problems with non-breaking spaces in Python. After some Googling I found [this](https://www.thelinuxrain.com/articles/how-to-deal-with-nbsps-in-a-terminal) post by \[Bob Mesibov\], and sure enough `ls -ls | grep $'\xc2\xa0'` highlighted the space in the filename (as shown in the header image of this post).

Renaming the file, and voila, `test` now works. 
