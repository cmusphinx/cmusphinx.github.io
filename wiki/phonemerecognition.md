---
layout: page 
title: Phoneme Recognition
---
## Phoneme Recognition (caveat emptor)

Frequently, people want to use Sphinx to do phoneme recognition.  In other words, they would like to convert speech to a stream of phonemes rather than words.  This is possible, although the results can be disappointing.  The reason is that automatic speech recognition relies heavily on contextual constraints (i.e. language modeling) to guide the search algorithm.  The phoneme recognition task is much less constrained that word decoding, and therefore the error rate (even when measured in terms of phoneme error for word decoding) is considerably higher.  For mostly the same reason, phoneme decoding is quite slow.

That said, even very inaccurate phoneme decoding can be helpful for diverse tasks including pronunciation modeling, speaker indentification, and voice conversion.
## Using pocketsphinx for phoneme recognition

Recently a support for phoneme recognition has been added to pocketsphinx decoder. To access described feature you need to checkout the latest code from subversion repository and build it from source. The release with this feature is also coming out soon.

Phoneme recognition is implemented as a separate search module like FSG or LM but it requires specific phone language model to understand possible phone frequencies. The model for US English is available in pocketsphinx distribution, it's ''pocketsphinx/model/en-us/en-us-phone.lm.dmp''. Phoneme recognition is enabled with ''-allphone phonetic.lm''.

For other languages you need a phonetic language model for your phoneset, steps are the following. You can take a text, convert it to a phonetic strings using the phonetic dictionary for your langauge. Just replace the words with their corresponding transcription. Since number of phones is small, text shouldn't be big either, just a book will do. If you have training data, you can use forced alignment to get transcription with dictionary variants. This way the phonetic transcription will be more precise. That you can build a language model from the phonetic transcription using any language model building tool like cmuclmtk or SRILM. The model will look like this:

	
	\data\
	ngram 1=35
	ngram 2=340
	ngram 3=1202
	
	\1-grams:
	-99.0000 `<s>`  0.0000
	-1.8779 AA      -2.3681
	-3.2104 AE      -1.1361
	-1.4280 AH      -2.4071
	-1.9864 AO      -2.2929
	-2.4635 AW      -1.8166
	-1.5254 AY      -2.3892
	..........


Now make sure you installed pocketsphinx properly and run the ''pocketsphinx_continuous'' program on any 16khz 16bit input file. For example take ''pocketsphinx/test/data/goforward.raw'' and lets decode it with en-us generic acoustic model 

	
	pocketsphinx_continuous -infile test/data/goforward.raw -hmm model/en-us/en-us \
	                        -allphone model/en-us/en-us-phone.lm.bin -backtrace yes \
	                        -beam 1e-20 -pbeam 1e-20 -lw 2.0


You should see a bunch of debugging output followed by a line that looks like this:

	
	000000000: SIL T OW F AO R W ER D T EH N M IY UW T ER Z S


That is your decoding result, you can also access individual phones and their times with pocketsphinx API.


## Training phonetic language model for decoding

To train phonetic language model you need to have the following:

 1.  Phonetic dictionary in your language
 2.  Sample text in your language (not very long, maybe 1000 lines)
 3.  Experience in scripting languages, say Python

Write a script, say in Python, to convert text to phonetic strings, basically replace every word in the text with corresponding phoneme sequence, you will get a text file with the list of sequences like this:

	
	SIL G OW F AO R W ER D T EH N M IY T ER Z S SIL
	SIL G OW F AO R W ER D S EH V EH N M IY T ER Z S SIL
	SIL S T AA P SIL
	SIL K W IH T SIL


Feed this text file into SRILM

	
	ngram-count -text la-phonetic-strings.txt -lm la-phone.lm


Use this lm in phonetic decoder.
