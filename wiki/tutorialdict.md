---
layout: page 
title: Building phonetic dictionary
---

### Introduction

Phonetic dictionary provides system the data to map vocabulary words to 
sequence of phonemes. It looks like this:

	
	hello H EH L OW
	world W ER L D


Dictionary can contain alternative pronunciations, in that case you can 
designate them with a number in parenthesis

	
	the TH IH
	the(2) TH AH


There are various phonesets to represent phones like IPA or SAMPA, CMUSphinx 
does not yet require you to use any well-known phoneset, moreover, it prefers 
to use letter-only phone names without special symbols. This requirement 
simplifies some processing algorithms, for example, you can create files with 
phone names.

Dictionary should contain all the words you are interested in, otherwise 
recognizer will not be able to recognize them. However, it is not sufficient to 
have the words in the dictionary, the recognizer looks for the word both in the 
dictionary and in the language model. Without language model the word will not 
be recognized even if you added it in the dictionary.

There is no need to remove unused words from the dictionary unless you want to 
save memory, extra words in the dictionary do not affect accuracy.

### Using existing dictionaries

There are number of dictionaries which cover languages we support - CMUDict for 
US English, French, German, Russian, Dutch, Italian, Spanish, Mandarin. Other 
dictionaries might be found on the web. If dictionary has proper format you can 
use it.

If dictionary does not cover all the words you are interested in you can extend 
it with g2p tool.


### Using G2P-seq2seq to extend the dictionary

There are various tools to help you to extend an existing dictionary for new 
words or to build a new dictionary from scratch: 
[Phonetisaurus](http://code.google.com/p/phonetisaurus), 
[Sequitur](http://www-i6.informatik.rwth-aachen.de/web/Software/g2p.html). 

We recommend to use our latest tool [g2p-seq2seq 
](https://github.com/cmusphinx/g2p-seq2seq). It is based on neural networks 
implemented in Tensorflow framework and provides a state of the art accuracy of 
conversion.

An English model 2-layer LSTM with 256 hidden units is [available for 
download]( 
https://sourceforge.net/projects/cmusphinx/files/G2P%20Models/g2p-seq2seq-cmudic
t.tar.gz/download ) on cmusphinx website. Unpack the model after download. It 
is trained on CMU English dictionary. Read my lips - this model works only for 
English. For other languages you need to bootstrap dictionary first as 
described below and then use G2P tool to extend it.

The easiest way to check how the tool works is to run it the interactive mode 
with model above and type the words

    g2p-seq2seq --interactive --model model_folder_path


    > hello
    HH EH L OW

To generate pronunciations for an English word list with a trained model, run

    g2p-seq2seq --decode your_wordlist --model model_folder_path

The wordlist is a text file with words, one word per line.

To train G2P you need a dictionary (word and phone sequence per line in 
standard form). To run the training

    g2p-seq2seq --train train_dictionary.dic --model model_folder_path

For more information on the tool see the corresponding page.

### Bootstrapping dictionary for other languages

If you do not have dictionary for your language there are usually several ways 
on how you can obtain them.

Usually dictionaries are bootstrapped with hand-written rules. You can find a 
list of phonemes for your language in Wikipedia page about your language and 
write a simple Python script to map words to phonemes. The best dictionary 
could not be covered with rules though, most languages have quite irregular 
pronunciation which might not be very obvious for newcomer even if it is 
conventionally thought you speak what is written. This is due to coarticulation 
effects in human speech. But for basic dictionary rules are sufficiently good 
enough.

You can crawl Wiktionary to get mapping for significant amount of words covered 
there.

You can use TTS tools like from [ OpenMary ](http://mary.dfki.de/ ) written in 
Java or from [Espeak](http://espeak.sourceforge.net) written in C to create the 
phonetic dictionary for the languages they support.

Many languages which use hieroglyphs like Korean or Japanese have specialized 
software like [Mecab](https://sourceforge.net/projects/mecab) to romanize their 
words. You can use Mecab to build a phonetic dictionary by converting words to 
romanized form and then simply applying rules to turn them into phones.

It is enough to transcribe few thousand most common words to bootstrap the 
dictionary.

Once dictionary is bootstrapped you can extend it to larger vocabulary with the 
g2p-seq2seq tool as described in previous chapter.
