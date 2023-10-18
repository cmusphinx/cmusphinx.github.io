---
layout: post
status: publish
published: true
title: PocketSphinx 5.0.0 release candidate 2
author:
  display_name: David Huggins-Daines
  email: dhdaines@gmail.com
author_email: dhdaines@gmail.com
excerpt_separator: <!--more-->
---

**Executive Summary: This is a release *candidate* and the API is not
yet stable so please don't package it.**

PocketSphinx now has a [release
candidate](https://github.com/cmusphinx/pocketsphinx/releases/tag/v5.0.0rc2).
You can also download it [from
PyPI](https://pypi.org/project/pocketsphinx5/).

Why release candidate 2?  Because there was a release candidate 1, but
it had various problems regarding installation, so I made another one.
This one is relatively complete, but the documentation isn't good, and
it hasn't been fully tested on Windows or Mac OS X.  If you are
courageous, you can try that.  Installation should be a matter of:

    cmake -S . -B build
    cmake --build build
    sudo cmake --build build --target install

The most important change versus `5prealpha` is, as mentioned
previously, the disappearance of `pocketsphinx_continuous` and the
"live" API in general, which has been replaced with
[`<pocketsphinx/endpointer.h>`](https://cmusphinx.github.io/doc/pocketsphinx/endpointer_8h.html).
The API is quite simple but it requires you to feed it data in precise
quantities.  The best way to do this is to ensure that you can read
data from a file stream, as shown in the examples
[`live.c`](https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live.c)
and
[`live.py`](https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live.py).

For command-line usage there is a very Unixy program called
`pocketsphinx` now, which nonetheless doesn't have a man page yet
(Update: it has a man page).  Use it like this:

    # From microphone
    sox -d $(pocketsphinx soxflags) | pocketsphinx
    # From file
    sox audio.wav $(pocketsphinx soxflags) | pocketsphinx

There are no innovations with respect to modeling, algorithms, etc,
and there will never be.  But I am trying to make this into a decent
piece of software nonetheless.  All documentation, bug reports (that
are actually bug reports and not just 'how do i run the program') and
such are welcome via
[GitHub](https://github.com/cmusphinx/pocketsphinx).
