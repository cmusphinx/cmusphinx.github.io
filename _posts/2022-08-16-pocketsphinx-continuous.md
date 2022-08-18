---
layout: post
status: publish
published: true
title: Why I Removed pocketsphinx_continuous And What You Can Do About It, Part One
author:
  display_name: David Huggins-Daines
  email: dhdaines@gmail.com
author_email: dhdaines@gmail.com
excerpt_separator: <!--more-->
---

**Executive Summary: Audio input is complicated and a speech
recognition engine, particularly a small one, should not be in the
business of handling it, particularly when
[sox](http://sox.sourceforge.net/) can do it for you.**

For most of recorded history, PocketSphinx installed a small program
called `pocketsphinx_continuous` which, among other things, would
record audio from the microphone and do speech recognition on it.  The
badly formatted comment at the top of the code explained exactly what
it was:


    * This is a simple example of pocketsphinx application that uses continuous listening
    * with silence filtering to automatically segment a continuous stream of audio input
    * into utterances that are then decoded.

Unfortunately, thought it was always intended as example code, because
it was installed as a program you can run, people (and I am one)
considered it to be the official command-line tool for PocketSphinx
and tried to build "applications" around it.  This usually ended in
frustration.  Why?

Time For Some Audio Theory!

Leaving aside the debatable usefulness of live-mode speech recognition
for tasks other hands-free automotive control (I don't care how big
the touchscreen is in your T\*sla, I don't want you touching it), it
is nonetheless an audio application.  But it is very much *not* like
your typical audio application.

If you, as a speech developer/user/ordinary human being, try to [read
the](https://jackaudio.org/api/)
[documentation](https://docs.microsoft.com/en-us/windows/win32/xaudio2/xaudio2-introduction)
[for
a](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
[typical audio
API](https://developer.apple.com/documentation/coreaudio) you are
likely to be deeply confused.  You do not care about latency.  You do
not want to create a processing graph with multiplex reverb units
chained into a multi-threaded non-blocking pipeline of
[HM-2s](https://en.wikipedia.org/wiki/Boss_HM-2).  You just want to
get a stream of PCM data, preferably 16kHz and 16-bit, from the
microphone.  How the h\*ck do you do that?  The documentation will not
help you, because the API will not help you either.  It is not written
for you, because audio hardware and software is not designed for you.

In the beginning, audio hardware on PCs existed for one reason: to
play games.  Later on, it was repurposed for recording and making
music.  Both of these applications have in common a single-minded
focus on minimizing latency.  When you jump on the boss monster's
head, it needs to go "splat" *right now* and not 100ms later.  When
you punch in the bass track, same thing (though I hope your bass
doesn't sound like the boss monster's head exploding).  As a
consequence, audio APIs do singularly un-useful things like making you
run your processing code in a separate real-time thread and only ever
feeding it 128 samples at a time.  (Speech processing uses frames that
are generally at least 400 samples long)

By contrast, while some speech applications like spoken dialogue care
deeply about latency, and while it's obviously good to have speech
recognition that runs faster than real-time and gives incremental
recognition results, by far the largest contributor to latency in
these systems is endpointing - i.e. *deciding when the user has
finally stopped speaking*, and this latency is at least two orders of
magnitude greater than what game and music developers are worried
about.  Also, endpointing (and speech processing in general) is a
language processing rather than an audio processing task.

All this is to say that handling audio input in a speech recognition
engine is super annoying and should be avoided if possible,
i.e. handled by some external library or program or other part of the
application code.  Ideally this external thing should, as noted above,
just provide a nicely buffered stream of plain old data in the optimal
format for the recognizer.

Luckily, there is a program like that, and it is so perfect that
development on it largely ceased **in 2015**.  Yes, I am talking about
good old [sox](http://sox.sourceforge.net/), the Sound eXchanger.
Think about it, would you rather:

 - Create a device context (in a platform-specific way)
 - Create a procesing thread (in a very platform-specific way)
 - Create a message queue or ring buffer sufficiently large to handle
   possibly slower than real-time processing (not knowing ahead of
   time how large this will be)
 - Write code to mix down the input, convert it to integers, and
   (maybe, though you don't have to) resample it to 16kHz
 - Spin up your processing thread possibly with real-time priority
 - Then, maybe, recognize some speech
 
Or:

    popen("sox -q -r 16000 -c 1 -b 16 -e signed-integer -d -t raw -");

And get some data with `fread()`?  From the point of view of someone
who has stepped up to minimally restart maintenance of what is
essentially abandonware, it's pretty clear which one I would prefer to
support.

So `pocketsphinx_continuous` (add `.exe` if you like) won't be coming
back.  At the moment, in Python, you can just do this:

```python
from pocketsphinx5 import Decoder
import subprocess
import os
MODELDIR = os.path.join(os.path.dirname(__file__), "model")
BUFSIZE = 1024

decoder = Decoder(
    hmm=os.path.join(MODELDIR, "en-us/en-us"),
    lm=os.path.join(MODELDIR, "en-us/en-us.lm.bin"),
    dict=os.path.join(MODELDIR, "en-us/cmudict-en-us.dict"),
)
sample_rate = int(decoder.config["samprate"])
soxcmd = f"sox -q -r {sample_rate} -c 1 -b 16 -e signed-integer -d -t raw -"
with subprocess.Popen(soxcmd.split(), stdout=subprocess.PIPE) as sox:
    decoder.start_utt()
    try:
        while True:
            buf = sox.stdout.read(BUFSIZE)
            if len(buf) == 0:
                break
            decoder.process_raw(buf)
    except KeyboardInterrupt:
        pass
    finally:
        decoder.end_utt()
    print(decoder.hyp().hypstr)
```

Or in C (see why we prefer to use Python? and no, C++ is NOT BETTER):

```c
#include <pocketsphinx.h>
#include <signal.h>

static int global_done = 0;
void
catch_sig(int signum)
{
    global_done = 1;
}

int
main(int argc, char *argv[])
{
    ps_decoder_t *decoder;
    cmd_ln_t *config;
    char *soxcmd;
    FILE *sox;
    #define BUFLEN 1024
    short buf[BUFLEN];
    size_t len;

    if ((config = cmd_ln_parse_r(NULL, ps_args(),
                                 argc, argv, TRUE)) == NULL)
        E_FATAL("Command line parse failed\n");
    ps_default_search_args(config);
    if ((decoder = ps_init(config)) == NULL)
        E_FATAL("PocketSphinx decoder init failed\n");
    #define SOXCMD "sox -q -r %d -c 1 -b 16 -e signed-integer -d -t raw -"
    len = snprintf(NULL, 0, SOXCMD,
                   (int)cmd_ln_float_r(config, "-samprate"));
    soxcmd = malloc(len + 1);
    if (signal(SIGINT, catch_sig) == SIG_ERR)
        E_FATAL_SYSTEM("Failed to set SIGINT handler");
    if (snprintf(soxcmd, len + 1, SOXCMD,
                 (int)cmd_ln_float_r(config, "-samprate")) != len)
        E_FATAL_SYSTEM("snprintf() failed");
    if ((sox = popen(soxcmd, "r")) == NULL)
        E_FATAL_SYSTEM("Failed to popen(%s)", soxcmd);
    free(soxcmd);
    ps_start_utt(decoder);
    while (!global_done) {
        len = fread(buf, sizeof(buf[0]), BUFLEN, sox);
        if (ps_process_raw(decoder, buf, len, FALSE, FALSE) < 0)
            E_FATAL("ps_process_raw() failed\n");
    }
    ps_end_utt(decoder);
    if (pclose(sox) < 0)
        E_ERROR_SYSTEM("Failed to pclose(sox)");
    if (ps_get_hyp(decoder, NULL) != NULL)
        printf("%s\n", ps_get_hyp(decoder, NULL));
    cmd_ln_free_r(config);
    ps_free(decoder);
        
    return 0;
}
```

What will come back for the release is a program which reads audio
from standard input and outputs recognition results in JSON, so you
can do useful things with them in another program.  This program will
probably be called `pocketsphinx`.  It will also do voice activity
detection, which will be the subject of the next in this series of
blog posts.  Obviously, if you want to build a real application,
you'll have to do something more sophisticated, probably a server, and
if I were you I would definitely write it in Python, though Node.js is
also a good choice and, hopefully, we will support it again for the
release.

Stay tuned!
