---
layout: page
title: Tuning speech recognition accuracy
---

---
* auto-gen TOC:
{:toc}
---

Speech recognition accuracy is not always great.

First, it is important to understand whether your accuracy is just lower than
expected or whether it is very low in general. If the accuracy is very low in
general, you most likely misconfigured the decoder. If it is lower than
expected, you can apply various ways to improve it.

The first thing you should do is to collect a database of test samples and
measure the recognition accuracy. You need to dump utterances into wav files,
write a reference text and use the decoder to decode it. Then calculate the
Word Error Rate (WER) using the `word_align.pl` tool from Sphinxtrain.
The size of the test database depends on the accuracy but usually it’s
sufficient to have 30 minutes of transcribed audio to test recognizer accuracy
reliably.

Only if you have a test database you can proceed with optimizing the recognition
accuracy.

## Reasons for bad accuracy

The top reasons for a bad accuracy are:

* The mismatch of the sample rate and the number of channels of the incoming
audio or the mismatch of the incoming audio bandwidth. It must be a 16 kHz
(or 8 kHz, depending on the training data), 16bit Mono (= single channel)
Little-Endian file.
You need to fix the sample rate of the source with resampling (only if its rate
is higher than that of the training data). You should not upsample a file and
decode it with acoustic models trained on audio with a higher sampling rate.
The audio file format (sampling rate, number of channels) can be verified using
the command

  ```
  sox --i /path/to/audio/file
  ```
Find more information here:
[What is sample rate?](/wiki/faq#q-what-is-sample-rate-and-how-does-it-affect-accuracy)

* The mismatch of the acoustic model. To verify this hypothesis you need to
construct a language model from the test database text. Such a language model
will be very good and must give you a high accuracy. If accuracy is still low,
you need to work more on the acoustic model. You can use acoustic model
adaptation to improve accuracy.

* The mismatch of the langauge model. You can create your own language model
to match the vocabulary you are trying to decode.

* The mismatch in the dictionary and the pronuncation of the words. In that
case some work must be done in the phonetic dictionary.

## Test database setup

To test the recognition you need to configure the decoding with the required
paramters, in particular, you need to have a language model `<your.lm>`. For
more details see the [Building a language model](/wiki/tutoriallm) page.

Create a *fileids* file `test.fileids`:

```
test1
test2
```

Create a transcription file `test.transcription`:

```
some text (test1)
some text (test2)
```

Put the audio files in the *wav* folder. Make sure those files have a proper
format and sample rate.

```
└─ wav
   ├─ test1.wav
   └─ test2.wav
```

## Running the test

Now, let’s run the decoder:

```bash
pocketsphinx_batch \
 -adcin yes \
 -cepdir wav \
 -cepext .wav \
 -ctl test.fileids \
 -lm `<your.lm>` \    # for example en-us.lm.bin from pocketsphinx
 -dict `<your.dic>` \ # for example cmudict-en-us.dict from pocketsphinx
 -hmm `<your_hmm>` \  # for example en-us
 -hyp test.hyp

word_align.pl test.transcription test.hyp
```

The `word_align.pl` script is a part of sphinxtrain distribution.

Make sure to add the `-samprate 8000` option to the above command if you are
decoding 8 kHz files!

The script `word-align.pl` from Sphinxtrain will report you the exact error rate
which you can use to decide if the adaptation worked for you. It will look
something like this:

```
TOTAL Words: 773 Correct: 669 Errors: 121
TOTAL Percent correct = 86.55% Error = 15.65% Accuracy = 84.35%
TOTAL Insertions: 17 Deletions: 11 Substitutions: 93
```

To see the speed of the decoding, check pocketsphinx logs, it should look like
this:

```
INFO: batch.c(761): 2484510: 9.09 seconds speech, 0.25 seconds CPU, 0.25 seconds wall
INFO: batch.c(763): 2484510: 0.03 xRT (CPU), 0.03 xRT (elapsed)
```

with `0.03 xRT` being a decoding speed ("0.03 times the recording time").

<span class="post-bottom-nav">
  [Training an acoustic model](/wiki/tutorialam)
  [Back to the Index](/wiki/tutorial)
</span>
