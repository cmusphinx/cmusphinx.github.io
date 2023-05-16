---
layout: post
status: publish
published: true
title: Live audio examples for Windows
author:
  display_name: David Huggins-Daines
  email: dhd@ecolingui.ca
author_email: dhd@ecolingui.ca
excerpt_separator: <!--more-->
---

Well, it turns out that people *were* using
[`pocketsphinx_continuous`](./2022-08-16-pocketsphinx-continuous.md),
at least sort of.  As I expected, they weren't really using the actual
`pocketsphinx_continuous` binary for anything useful other than
recognizing from files.  But, well, the code *did* claim to be example
code, and so obviously people were using it ... as example code.

...which is a perfectly sensible thing to do, and unfortunately in
removing the audio support from PocketSphinx, it became considerably
less useful as an example of how to do recognition from a microphone,
particularly if the solution of running
[SoX](https://sox.sourceforge.net/) in a subprocess isn't an appealing
one (as on Windows, for instance).

The sensible solution to this is to bring back something like
`pocketsphinx_continuous` but explicitly in the form of example code.
Adding cross-platform audio support to the library is absolutely
something I will not do, but there are some other options,
[PortAudio](https://portaudio.com) foremost among them.  So, here is
an example of using PortAudio:

https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_portaudio.c

That said, wrangling external dependencies on Windows is very
annoying.  To use the above example may require a certain amount of
path and environment wrangling to get CMake/VSCode/Visual Studio to
find PortAudio.  For this reason there is also now an example of using
the Win32 Waveform Audio API directly:

https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_win32.c

Note that in both cases you may have quite bad results when running a
"Debug" build, because Windows is very slow, and Visual C++ outputs
extremely slow code when debugging is enabled.

These examples are included in the upcoming 5.0.1 release in the
`examples` directory.
