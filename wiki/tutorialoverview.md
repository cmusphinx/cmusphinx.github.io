---
layout: page
title: Overview of the CMUSphinx toolkit
---

The CMUSphinx toolkit is a venerable speech recognition toolkit with
various tools used to build speech applications. We are currently
trying to consolidate it to reduce confusion.  To that end, these are
the currently maintained components:

* Pocketsphinx — lightweight recognizer library written in C
* Sphinxtrain — acoustic model training tools

We recommend that you use the latest development code:

* [pocketsphinx](https://github.com/cmusphinx/pocketsphinx)
* [sphinxtrain](https://github.com/cmusphinx/sphinxtrain)

Of course, many things are missing. Things like building a phonetic
model capable of handling an infinite vocabulary, postprocessing of the
decoding result, sense extraction and other semantic tools should be
added one day. Probably you should take it on.

The following resources are the main ones for CMUSphinx developers:

* [Website](http://cmusphinx.github.io)
* [Github](https://github.com/cmusphinx)
* [Telegram Chat](https://t.me/speech_recognition_help)

<span class="post-bottom-nav">
  [Basic concepts of speech recognition](/wiki/tutorialconcepts)
  [Before you start](/wiki/tutorialbeforestart/)
</span>
