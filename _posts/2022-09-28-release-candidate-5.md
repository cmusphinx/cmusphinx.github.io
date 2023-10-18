---
layout: post
status: publish
published: true
title: PocketSphinx 5.0.0 release candidate 5
author:
  display_name: David Huggins-Daines
  email: dhdaines@gmail.com
author_email: dhdaines@gmail.com
excerpt_separator: <!--more-->
---

**Executive Summary: Please try this one, there won't be another.**

Yes, it's that time of week again, time for another [release
candidate](https://github.com/cmusphinx/pocketsphinx/releases/tag/v5.0.0rc5)
You can also download it [from
PyPI](https://pypi.org/project/pocketsphinx/).

There are a lot of changes so I suggest you look at the release notes
at the link above.  Python code should continue to work as before,
though you may get some deprecated warnings when you try to use the
inappropriately named `set_{fsg,lm,kws}` methods.  Don't use them,
they have the wrong names, use `add_*` instead.  The names were
changed because **they don't set anything** and you have to actually
activate the search module afterwards.  Now you use
`ps_activate_search()` to do that, and not `ps_set_search()`, because
this, too, is a much better name.

That's actually the least of it.  The big news is that force-alignment
and subword alignment are now quite doable, from the command-line,
from the C API, and from Python.  There are some tests and examples
for you to look at.

The last known portability issue (which was actually, like, a bug) is
fixed and you won't get unpredictable and bad results on MIPS systems.
There are surely others, though.  Ideally our CI testing would run
things on various emulators, but it's slow and unwieldy to do that.

The JSGF compiler is back to producing unreasonable numbers of epsilon
transitions ("null" transitions for the less FST-aware), but it
produces correct output now.

Pull requests and bug reports and such are welcome via
[GitHub](https://github.com/cmusphinx/pocketsphinx).
