---
layout: page
title: Building an application with PocketSphinx
---

---
* auto-gen TOC:
{:toc}
---

In this tutorial we will walk through a simple code example in C using
PocketSphinx.  This corresponds exactly to the [`live_portaudio.c`
example](https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_portaudio.c)
in the source code.  So, TL;DR, you could just try to compile that:

    cmake -G Ninja -S. -B build
    cmake --build --target live_portaudio

## Building PocketSphinx

First, obtain the source code, either by downloading a release from
the [GitHub releases
page](https://github.com/cmusphinx/pocketsphinx/releases) or by
[cloning the source with git](https://github.com/cmusphinx/pocketsphinx.git)

For more details see the [download page](/wiki/download) or the
[README
file](https://github.com/cmusphinx/pocketsphinx/blob/master/README.md)

PocketSphinx uses [CMake](https://cmake.org/) to manage configuration
and buliding on multiple platforms.  By default, it builds static
libraries and binaries which can simply be used from the source
directory.  You may also install it system-wide or in a user
directory.  In the case where it is installed, it will use
[`pkg-config`](https://www.freedesktop.org/wiki/Software/pkg-config/)
to allow you to find library and header directories and names, but
this isn't really necessary, since there is just one header file
([`<pocketsphinx.h>`](../doc/pocketsphinx/pocketsphinx_8h.html)) and
one library (`-lpocketsphinx`).

For the purposes of this tutorial we will install it in a local
directory.

### Installation on a Unix-like system (including MacOS)

Make sure you have CMake, PortAudio, and a working C compiler
installed.  On GNU/Linux you can use your package manager, whatever it
is.  For example on Ubuntu/Debian/etc:

    sudo apt install build-essential cmake ninja-build portaudio19-dev

On MacOS this will require you to minimally install the "Xcode
command-line tools".  If you have installed
[Homebrew](https://brew.sh/) then you have these already.  You can
then install CMake, either with the [official
installer](https://github.com/Kitware/CMake/releases/download/v3.28.1/cmake-3.28.1-macos-universal.dmg),
or [from Homebrew](https://formulae.brew.sh/formula/cmake#default).
For PortAudio... I dunno, use Homebrew, I guess:

    brew install cmake portaudio

We will assume that you install PocketSphinx in directory called
`cmusphinx` inside your home directory.  It should be as simple as
(assuming CMake is installed):

    cmake -S . -B build -DCMAKE_INSTALL_PREFIX=$HOME/cmusphinx
    cmake --build build --target install

If you are lucky enough to have [Ninja](https://ninja-build.org/)
installed, you can make the build many times faster (it takes 1.7
seconds to complete on an [Intel processor from
2009](https://www.intel.com/content/www/us/en/products/sku/41316/intel-core-i7860-processor-8m-cache-2-80-ghz/specifications.html)),
simply add `-G Ninja` to the first line above.

### Windows

Of course, Windows is more complicated in every possible way.  There
are many ways to do everything, all of them inconvenient and
frustrating.  The path of least resistance is to just install
[MSYS2](https://www.msys2.org/), start the MSYS2 shell (preferably the
`UCRT64` version whatever that is) then install CMake, PortAudio and
Ninja along with the compiler:

    pacman -S mingw-w64-ucrt-x86_64-gcc cmake ninja mingw-x64-ucrt-x86_64-portaudio

Now build:

    cmake -S . -B build -G Ninja -DCMAKE_INSTALL_PREFIX=$HOME/cmusphinx
    cmake --build build --target install
    
Note that because you're running the MSYS2 version of CMake, you
*cannot* directly use a Windows-y path like `$LOCALAPPDATA/cmusphinx`
or (horror) `%LOCALAPPDATA%/cmusphinx` as the `CMAKE_INSTALL_PREFIX`,
as it [won't understand the drive
letters](https://www.msys2.org/docs/filesystem-paths/).  You can use
`cygpath` for this, e.g.:

    cmake -S . -B build -G Ninja -DCMAKE_INSTALL_PREFIX=$(cygpath $LOCALAPPDATA)/cmusphinx
    cmake --build build --target install

## Configuration

Now that you have installed PocketSphinx you must set one little
environment variable to make sure that it can find its models.  When
you ran the first `cmake` command above you may have seen a line like
this:

    MODELDIR="/home/user/cmusphinx/share/pocketsphinx/model"

You'll need to set `POCKETSPHINX_PATH` to this directory.  In the near
future the install target will also tell you about this, and will also
create a little script to do it for you, but for now you have to do it
manually:

    export POCKETSPHINX_PATH=$HOME/cmusphinx/share/pocketsphinx/model

## Using the Pocketsphinx API

Okay, let's get to the code!  As a reminder, you can see the whole
thing at
https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_portaudio.c

### General Principles

The reference documentation for the API is available at
<https://cmusphinx.github.io/doc/pocketsphinx/>.  There are three
opaque types (like classes) that we will create and use to configure
the recognizer, detect speech segments in the input, and recognize
speech:

- [`ps_config_t`](../doc/pocketsphinx/structps__config__t.html)
- [`ps_endpointer_t`](../doc/pocketsphinx/structps__endpointer__t.html)
- [`ps_decoder_t`](../doc/pocketsphinx/structps__decoder__t.html)

In general, PocketSphinx types have a function called `TYPE_init`
which creates an instance of type and a function called `TYPE_free`
which releases an instance of a type.  The general rule is that if you
create an instance in your code, you will always need to free it, and
if you didn't create it (i.e. it was returned to you by some API
function), you shouldn't free it.  The sole exception to this is
iterator types like
[`ps_seg_t`](../doc/pocketsphinx/structps__seg__t.html) and
[`ps_alignment_iter_t`](../doc/pocketsphinx/structps__alignment__iter__t.html),
which must be "freed" if you stop iterating over them before the end
of the list.

That sounds confusing but it makes sense if you think about it!  When
in doubt, remember: ["Memory leaks are quite acceptable in many
applications"](https://accu.org/journals/overload/1/2/toms_1356/)

### Initialization

First we will [create the
`ps_config_t`](https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_portaudio.c#L52C1-L54C1)
with the set of default arguments, which also includes a default
acoustic and language model:

    config = ps_config_init(NULL);
    ps_default_search_args(config);

The `ps_config_t` has a bunch of associated functions to get
information out of it.  We will use one of these in particular to
obtain the sampling rate, which we will [use to initialize
PortAudio](https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_portaudio.c#L65)
later on.

(note that you can *also* do this the other way around, and use the
sampling rate provided by your audio stream to initialize the
recognizer, which makes more sense in some cases)

Skipping the details of initializing PortAudio, we will now
[initialize the
recognizer](https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_portaudio.c#L58),
which is called a "decoder" for Reasons:

    if ((decoder = ps_init(config)) == NULL)
        E_FATAL("PocketSphinx decoder init failed\n");

And [the
endpointer](https://github.com/cmusphinx/pocketsphinx/blob/master/examples/live_portaudio.c#L60C1-L62C1),
which is the component that detects speech in the audio stream.  Yes,
that's a lot of zeros.  You can see what they mean in the
documentation:

    if ((ep = ps_endpointer_init(0, 0.0, 0, 0, 0)) == NULL)
        E_FATAL("PocketSphinx endpointer init failed\n");

### Endpointing

The endpointer works by consuming a "frame" of audio data and
returning a "frame" of speech data, if any was detected.  Again, since
we're in C here, it's important to remember that:

- The input data is owned by the caller
- The output data is owned by the endpointer

What this means is that you can simply allocate a single buffer for
input audio and reuse it through your whole application, and that you
should never do *anything* with the frames of speech returned by the
endpointer except:

- Pass them directly to the decoder
- Copy them somewhere else if you want to save them

This means, by the way, that you should also never try to share an
endpointer or a decoder between multiple threads.

How do you know what to allocate?  The endpointer tells you, with
`ps_endpointer_frame_size`:

    frame_size = ps_endpointer_frame_size(ep);
    if ((frame = malloc(frame_size * sizeof(frame[0]))) == NULL)
        E_FATAL_SYSTEM("Failed to allocate frame");

If you don't care about compatibility with ancient C compilers you can
simply do this and not worry about freeing anything later on:

    int16_t frame[ps_endpointer_frame_size(ep)];
    
(as an aside, likely a future version of PocketSphinx will drop C89
compatibility entirely, since it is dubiously C89-compliant already)

It's important to note that you can *only* ever pass buffers of this
size to the endpointer.  With certain low-level audio APIs, this means
that you'll have to do some buffering.  Luckily, PortAudio allows you
to request the buffer size, which we do when opening the audio stream:

    if ((err = Pa_OpenDefaultStream(&stream, 1, 0, paInt16,
                                    ps_config_int(config, "samprate"),
                                    frame_size, NULL, NULL)) != paNoError)
        E_FATAL("Failed to open PortAudio stream: %s\n",
                Pa_GetErrorText(err));

Now, your application will wait to get some audio buffers, which
usually involves either a callback function (yuck) or a nice, simple
loop (hooray).  Luckily, PortAudio gives us the second option.  So, we
sit in a loop, which looks like this:

1. Check if we are currently in a speech section with
   `ps_endpointer_in_speech`.
2. Wait for an audio buffer.
3. Pass it to the endpointer with `ps_endpointer_process`.  If it's
   `NULL`, then go back to step 1.
4. If we weren't in a speech section before, start recognizing speech.
5. Pass the speech buffer to the recognizer.
6. If we are no longer in a speech section (check with
   `ps_endpointer_in_speech` again), stop recognizing speech and get
   recognition results.
   
### Processing

Before we can recognize any speech, we need to start an "utterance" by
calling `ps_start_utt`.  Then we can pass buffers of audio (of any
size, in this case) using `ps_process_raw`.  This has a couple of
options which we won't use here for live-mode recognition, but may be
useful in other cases - one can instruct it to simply buffer the audio
without actually doing any recognition (in the case of a very slow
computer), or to treat the entire buffer as a single utterance (useful
when recognizing entire files at once, as it gives better accuracy).

### Getting Results

Recognition results can be requested using `ps_get_hyp` at any point
between a call to `ps_start_utt` and `ps_end_utt`.  This function
simply returns a string - if you want a word segmentation, you can use
`ps_seg_iter`.

As with speech buffers, you didn't allocate this string, you shouldn't
free it, and you shouldn't do anything with it except:

- Print it out or some other immediate action.
- Copy it if you wish to save or store it for later use.

Likewise same warnings about not sharing `ps_decoder_t` between threads.

### Code

Concretely, the whole thing looks like this:

    while (!global_done) {
        const int16 *speech;
        int prev_in_speech = ps_endpointer_in_speech(ep);
        if ((err = Pa_ReadStream(stream, frame, frame_size)) != paNoError) {
            E_ERROR("Error in PortAudio read: %s\n",
                Pa_GetErrorText(err));
            break;
        }
        speech = ps_endpointer_process(ep, frame);
        if (speech != NULL) {
            const char *hyp;
            if (!prev_in_speech)
                ps_start_utt(decoder);
            if (ps_process_raw(decoder, speech, frame_size, FALSE, FALSE) < 0)
                E_FATAL("ps_process_raw() failed\n");
            if (!ps_endpointer_in_speech(ep)) {
                ps_end_utt(decoder);
                if ((hyp = ps_get_hyp(decoder, NULL)) != NULL) {
                    printf("%s\n", hyp);
                    fflush(stdout);
                }
            }
        }
    }

How to compile?  Well... the actual example of course can just be
built with CMake as noted way up at the top, but for your own code
you'll need to know how to compile and link it.  If you installed
using the commands above you'll find the PocketSphinx headers in
`$HOME/cmusphinx/include` and the libraries in `$HOME/cmusphinx/lib`.
For PortAudio, they'll be... somewhere, but `pkg-config` can tell you.
So, if your code was in `example.c`:

    cc -o example example.c \
        -I$HOME/cmusphinx/include -L$HOME/cmusphinx/lib -lpocketsphinx -lm \
        $(pkg-config --static --libs --cflags portaudio-2.0)

## Advanced usage

For more advanced uses of the API please check the API reference.

* For word segmentations, the API provides an iterator object which is
  used to iterate over the sequence of words. This iterator object is an
  abstract type, with some accessors provided to obtain timepoints, scores and,
  most interestingly, posterior probabilities for each word.
* The confidence of the whole utterance can be accessed with the `ps_get_prob` method.
* You can access the lattice if needed.
* You can configure multiple searches and switch between them in runtime.

### Searches

As a developer you can configure several "search" objects with different
grammars and language models and switch between them during runtime to provide
interactive experience for the user.

There are multiple possible search modes:

* *keyword*: efficiently looks for a keyphrase and ignores other speech.
  It Allows to configure the detection threshold.
* *grammar*: recognizes speech according to the JSGF grammar. Unlike keyphrase
  search, grammar search doesn’t ignore words which are not in the grammar but
  tries to recognize them.
* *ngram/lm*: recognizes natural speech with a language model.
* *allphone*: recognizes phonemes with a phonetic language model.

Each search has a name and can be referenced by a name. Names are
application-specific. The function `ps_set_search` allows to activate the
search that was previously added by a name.

In order to add a search, one needs to point to the grammar/language model
describing the search. The location of the grammar is specific to the application.
If only a simple recognition is required it is sufficient to add a single search
or to just configure the required mode using configuration options.

The exact design of a search depends on your application. For example, you
might want to listen for an activation keyword first and once this keyword is
recognized switch to ngram search to recognize the actual command. Once you
recognized the command you can switch to grammar search to recognize the
confirmation and then switch back to keyword listening mode to wait for another
command.

<span class="post-bottom-nav">
  [Building an application with Sphinx4](/wiki/tutorialsphinx4)
  [Using PocketSphinx on Android](/wiki/tutorialandroid)
</span>
