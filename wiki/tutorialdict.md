---
layout: page
title: Building a phonetic dictionary
---

---
* auto-gen TOC:
{:toc}
---

### Introduction

A phonetic dictionary provides the system with a mapping of vocabulary words
to sequences of phonemes. It might look like this:

	hello H EH L OW
	world W ER L D

A dictionary can also contain alternative pronunciations. In that case you can
designate them with a number in parentheses:

	the TH IH
	the(2) TH AH

There are various phonesets to represent phones, such as
[IPA](https://en.wikipedia.org/wiki/International_Phonetic_Alphabet) or
[SAMPA](https://en.wikipedia.org/wiki/Speech_Assessment_Methods_Phonetic_Alphabet).
CMUSphinx does not yet require you to use any well-known phoneset, moreover, it
prefers to use letter-only phone names without special symbols. This requirement
simplifies some processing algorithms, for example, you can create files with
phone names as part of the filenames without any violating of the OS filename
requirements.

A dictionary should contain all the words you are interested in, otherwise
the recognizer will not be able to recognize them. However, it is not sufficient
to have the words in the dictionary. The recognizer looks for a word in both
the dictionary and the language model. Without the language model, a word will
not be recognized, even if it is present in the dictionary.

There is no need to remove unused words from the dictionary unless you want to
save memory, extra words in the dictionary do not affect accuracy.


### Using existing dictionaries

There are a number of dictionaries which cover languages we support – [CMUDict](https://github.com/cmusphinx/cmudict)
for US English, French, German, Russian, Dutch, Italian, Spanish and Mandarin.
Other dictionaries might be found on the web. If a dictionary has a proper
format you can use it.

If a dictionary does not cover all the words you are interested in, you can
extend it with the *g2p* tool.


### Using g2p-seq2seq to extend the dictionary

There are various tools to help you to extend an existing dictionary for new
words or to build a new dictionary from scratch. Two of them are
[Phonetisaurus](http://code.google.com/p/phonetisaurus) and
[Sequitur](http://www-i6.informatik.rwth-aachen.de/web/Software/g2p.html).

We recommend to use our latest tool [g2p-seq2seq
](https://github.com/cmusphinx/g2p-seq2seq). It is based on neural networks
implemented in the Tensorflow framework and provides a state-of-the-art accuracy
of conversion.

An English model 2-layer LSTM with 512 hidden units is [available for
download](https://sourceforge.net/projects/cmusphinx/files/G2P%20Models/g2p-seq2seq-cmudict.tar.gz/download) on the CMUSphinx website. Unpack the model after downloading. It
is trained on the CMU English dictionary. As the name says, this model works
only for English. For other languages you first need to bootstrap a dictionary
as described below and then use the G2P tool to extend it.

The easiest way to check how the G2P tool works is to run the interactive
mode with the model from above:

    g2p-seq2seq --interactive --model model_folder_path

    > hello
    HH EH L OW

To generate pronunciations for an English word list with a trained model, run:

    g2p-seq2seq --decode your_wordlist --model model_folder_path

The wordlist is a text file with words, one word per line.

To train G2P you need a dictionary (a word and phone sequence per line in
the standard form). Run the training with:

    g2p-seq2seq --train train_dictionary.dic --model model_folder_path

For more information on the G2P tool have a look at the [Readme of the G2P project](https://github.com/cmusphinx/g2p-seq2seq).

### Bootstrapping a dictionary for other languages

If you do not have a dictionary for your language there are usually several ways
how you can create them.

Usually, dictionaries are bootstrapped with hand-written rules. You can find a
list of phonemes for your language in the Wikipedia page about your language and
write a simple Python script to map words to phonemes. The best dictionary
could not be covered with rules though, most languages have quite irregular
pronunciation which might not be very obvious for a newcomer even if it is
conventionally thought that you speak what is written. This is due to
coarticulation effects in human speech. However, for a basic dictionary, rules
are sufficiently good enough.

You can crawl the Wiktionary to get a mapping for a significant amount of words
covered there.

You can use TTS tools like from [OpenMary](http://mary.dfki.de/) written in
Java or from [Espeak](http://espeak.sourceforge.net) written in C to create the
phonetic dictionary for the languages they support.

Many languages which use hieroglyphs like Korean or Japanese have specialized
software like [Mecab](https://sourceforge.net/projects/mecab) to romanize their
words. You can use Mecab to build a phonetic dictionary by converting words to
the romanized form and then simply applying rules to turn them into phones.

It is enough to transcribe a few thousand most common words to bootstrap the
dictionary.

Once your dictionary is bootstrapped you can extend it to hold a larger
vocabulary with the g2p-seq2seq tool as described in the previous section.

<span class="post-bottom-nav">
  [Using PocketSphinx on Android](/wiki/tutorialandroid)
  [Building a language model](/wiki/tutoriallm)
</span>
