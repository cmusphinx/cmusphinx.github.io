---
layout: page
title: Adapting the default acoustic model
---

---
* auto-gen TOC:
{:toc}
---

> **Caution!**  
  This tutorial uses the [5 pre-alpha release](https://sourceforge.net/projects/cmusphinx/files/sphinx4/5prealpha/).  
  It is not going to work for older versions.

This page describes how to do some simple acoustic model adaptation to improve
speech recognition in your configuration. Please note that the adaptation
doesn't necessary adapt for a particular speaker. It just improves the fit
between the adaptation data and the model. For example you can adapt to your
own voice to make dictation good, but you also can adapt to your particular
recording environment, your audio transmission channel, your accent or accent
of your users. You can use a model trained with clean broadcast data and
telephone data to produce a telephone acoustic model by doing adaptation.
Cross-language adaptation also make sense, you can for example adapt an English
model to sounds of another language by creating a phoneset map and creating
another language dictionary with an English phoneset.

The adaptation process takes transcribed data and improves the model you
already have. It’s more robust than training and could lead to good results
even if your adaptation data is small. For example, it’s enough to have 5
minutes of speech to significantly improve the dictation accuracy by adapting
to the particular speaker.

The methods of adaptation are a bit different between PocketSphinx and Sphinx4
due to the different types of acoustic models used. For more technical
information on that read the article about
[Acoustic Model Types](/wiki/acousticmodeltypes).

##  Creating an adaptation corpus

The first thing you need to do is to create a corpus of adaptation data. The
corpus will consist of

* a list of sentences
* a dictionary describing the pronunciation of all the words in that list of sentences
* a recording of you speaking each of those sentences

### Required files

The actual set of sentences you use is somewhat arbitrary, but ideally it
should have good coverage of the most frequently used words or phonemes in the
set of sentences or the type of text you want to recognize. For example, if you
want to recognize isolated commands, you need tor record them. If you want to
recognize dictation, you need to record full sentences. For simple voice
adaptation we have had good results simply by using sentences from the [CMU
ARCTIC](http://festvox.org/cmu_arctic/) text-to-speech databases. To that
effect, here are the first 20 sentences from ARCTIC, a `.fileids` file, and a
transcription file:

* [arctic20.fileids](http://cmusphinx.github.io/data/arctic20.fileids)
* [arctic20.transcription](http://cmusphinx.github.io/data/arctic20.transcri
ption)

The sections below will refer to these files, so, if you want to follow along we
recommend downloading these files now. You should also make sure that you have
downloaded and compiled sphinxbase and sphinxtrain.

### Recording your adaptation data

In case you are adapting to a single speaker you can record the adaptation data
yourself. This is unfortunately a bit more complicated than it ought to be.  
Basically, you need to record a single audio file for each sentence in the
adaptation corpus, naming the files according to the names listed in
`arctic20.transcription` and `arctic20.fileids`.

In addition, you need to make sure that you record at a *sampling rate of
16 kHz* (or 8 kHz if you adapt a telephone model) in *mono with a single channel*.

The simplest way would be to start a sound recorder like Audacity or Wavesurfer
and read all sentences in one big audio file. Then you can cut the audio files
on sentences in a text editor and make sure every sentence is saved in the
corresponding file. The file structure should look like this:

	arctic_0001.wav  
	arctic_0002.wav
	.....
	arctic_0019.wav
	arctic20.fileids
	arctic20.transcription

You should verify that these recordings sound okay. To do this, you can play
them back with:

```bash
for i in *.wav; do play $i; done
```

If you already have a recording of the speaker, you can split it on sentences and
create the `.fileids` and the `.transcription` files.

If you are adapting to a channel, accent or some other generic property of the
audio, then you need to collect a little bit more recordings manually. For
example, in a call center you can record and transcribe hundred calls and use
them to improve the recognizer accuracy by means of adaptation.

## Adapting the acoustic model

First we will copy the default acoustic model from PocketSphinx into the
current directory in order to work on it. Assuming that you installed
PocketSphinx under `/usr/local`, the acoustic model directory is
`/usr/local/share/pocketsphinx/model/en-us/en-us`. Copy this directory to
your working directory:

	cp -a /usr/local/share/pocketsphinx/model/en-us/en-us .

Let’s also copy the dictionary and the langauge model for testing:

	cp -a /usr/local/share/pocketsphinx/model/en-us/cmudict-en-us.dict .
	cp -a /usr/local/share/pocketsphinx/model/en-us/en-us.lm.bin .

### Generating acoustic feature files

In order to run the adaptation tools, you must generate a set of acoustic model
feature files from these WAV audio recordings. This can be done with the
`sphinx_fe` tool from SphinxBase. It is imperative that you make sure you
are using the same acoustic parameters to extract these features as were used
to train the standard acoustic model. Since PocketSphinx 0.4, these are stored
in a file called `feat.params` in the acoustic model directory. You can
simply add it to the command line for `sphinx_fe`, like this:

	sphinx_fe -argfile en-us/feat.params \
	        -samprate 16000 -c arctic20.fileids \
	       -di . -do . -ei wav -eo mfc -mswav yes

You should now have the following files in your working directory:

	en-us
	arctic_0001.mfc
	arctic_0001.wav
	arctic_0002.mfc
	arctic_0002.wav
	arctic_0003.mfc
	arctic_0003.wav
	.....
	arctic_0020.wav
	arctic20.fileids
	arctic20.transcription
	cmudict-en-us.dict
	en-us.lm.bin

### Converting the sendump and mdef files

Some models like en-us are distributed in compressed version. Extra files
that are required for adaptation are excluded to save space. For the en-us model
from pocketsphinx you can download the full version suitable for adaptation:

[cmusphinx-en-us-ptm-5.2.tar.gz
](http://sourceforge.net/projects/cmusphinx/files/Acoustic%20and%20Language%20Mo
dels/US%20English%20Generic%20Acoustic%20Model/cmusphinx-en-us-ptm-5.2.tar.gz/do
wnload)

Make sure you are using the full model with the `mixture_weights` file present.

If the `mdef` file inside the model is converted to binary, you will also need
to convert the `mdef` file from the acoustic model to the plain text format used
by the SphinxTrain tools. To do this, use the `pocketsphinx_mdef_convert`
program:

	pocketsphinx_mdef_convert -text en-us/mdef en-us/mdef.txt

In the downloads the `mdef` is already in the text form.

### Accumulating observation counts

The next step in the adaptation is to collect statistics from the adaptation data.  
This is done using the `bw` program from SphinxTrain. You should be able to find
the `bw` tool in a sphinxtrain installation in the folder
`/usr/local/libexec/sphinxtrain` (or under another prefix on Linux) or in
`bin\Release` (in the sphinxtrain directory on Windows). Copy it to the working
directory along with the `map_adapt` and `mk_s2sendump` programs.

Now, to collect the statistics, run:

	./bw \
	 -hmmdir en-us \
	 -moddeffn en-us/mdef.txt \
	 -ts2cbfn .ptm. \
	 -feat 1s_c_d_dd \
	 -svspec 0-12/13-25/26-38 \
	 -cmn current \
	 -agc none \
	 -dictfn cmudict-en-us.dict \
	 -ctlfn arctic20.fileids \
	 -lsnfn arctic20.transcription \
	 -accumdir .

Make sure the arguments in the `bw` command match the parameters in the
`feat.params` file inside the acoustic model folder. Please note that not all
the parameters from `feat.param` are supported by `bw`.
`bw` for example doesn't suppport `upperf` or other feature extraction
parameters. You only need to use parameters which are accepted, other parameters
from `feat.params` should be skipped.

For example, for a continuous model you don't need to include the `svspec`
option. Instead, you need to use just `-ts2cbfn .cont.` For semi-continuous
models use `-ts2cbfn .semi`. If the model has a `feature_transform` file like
the en-us continuous model, you need to add the `-lda feature_transform`
argument to `bw`, otherwise it will not work properly.

If you are missing the `noisedict` file, you also need an extra step. Copy
the `fillerdict` file into the directory that you choose in the `hmmdir`
parameter and renaming it to `noisedict`.

### Creating a transformation with MLLR

MLLR transforms are supported by pocketsphinx and sphinx4. MLLR is a cheap
adaptation method that is suitable when the amount of data is limited. It’s a
good idea to use MLLR for online adaptation. MLLR works best for a continuous
model. Its effect for semi-continuous models is very limited since
semi-continuous models mostly rely on mixture weights. If you want the best
accuracy you can combine MLLR adaptation with MAP adaptation below. On the other
hand, because MAP requires a lot of adaptation data it is not really practical
to use it for continuous models. For continuous models MLLR is more reasonable.

Next, we will generate an MLLR transformation which we will pass to the decoder
to adapt the acoustic model at run-time. This is done with the `mllr_solve`
program:

	./mllr_solve \
	    -meanfn en-us/means \
	    -varfn en-us/variances \
	    -outmllrfn mllr_matrix -accumdir .

This command will create an adaptation data file called `mllr_matrix`. Now,
if you wish to decode with the adapted model, simply add `-mllr mllr_matrix`
(or whatever the path to the mllr_matrix file you created is) to your
pocketsphinx command line.

### Updating the acoustic model files with MAP

MAP is a different adaptation method. In this case, unlike for MLLR, we don’t
create a generic transform but update each parameter in the model. We will now
copy the acoustic model directory and overwrite the newly created directory
with the adapted model files:

	cp -a en-us en-us-adapt

To apply the adaptation, use the `map_adapt` program:

	./map_adapt \
	    -moddeffn en-us/mdef.txt \
	    -ts2cbfn .ptm. \
	    -meanfn en-us/means \
	    -varfn en-us/variances \
	    -mixwfn en-us/mixture_weights \
	    -tmatfn en-us/transition_matrices \
	    -accumdir . \
	    -mapmeanfn en-us-adapt/means \
	    -mapvarfn en-us-adapt/variances \
	    -mapmixwfn en-us-adapt/mixture_weights \
	    -maptmatfn en-us-adapt/transition_matrices

### Recreating the adapted sendump file

If you want to save space for the model you can use a `sendump` file which is
supported by PocketSphinx. For Sphinx4 you don’t need that. To recreate the
`sendump` file from the updated `mixture_weights` file run:

	./mk_s2sendump \
	    -pocketsphinx yes \
	    -moddeffn en-us-adapt/mdef.txt \
	    -mixwfn en-us-adapt/mixture_weights \
	    -sendumpfn en-us-adapt/sendump

Congratulations! You now have an adapted acoustic model.

The `en-us-adapt/mixture_weights` and `en-us-adapt/mdef.txt` files are not used
by the decoder, so, if you like, you can delete them to save some space.

## Other acoustic models

For Sphinx4, the adaptation is the same as for PocketSphinx, except that
Sphinx4 can not read the binary compressed `mdef` and `sendump` files, you need
to leave the `mdef` and the `mixture weights` file.

## Testing the adaptation

After you have done the adaptation, it’s critical to test the adaptation
quality. To do that you need to setup the database similar to the one used for
adaptation. To test the adaptation you need to configure the decoding with the
required paramters, in particular, you need to have a language model
`<your.lm>`. For more details see the tutorial on
[Building a Language Model](/wiki/tutoriallm). The detailed process of testing
the model is covered in [another part of the tutorial](/wiki/tutorialtuning).

You can try to run the decoder on the original acoustic model and on the new
acoustic model to estimate the improvement.

## Using the model

After adaptation, the acoustic model is located in the folder `en-us-adapt`.
You need only that folder. The model should have the following files:

	mdef
	feat.params
	mixture_weights
	means
	noisedict
	transition_matrices
	variances

depending on the type of the model you trained.

To use the model in PocketSphinx, simply put the model files to the resources
of your application. Then point to it with the `-hmm` option:

	pocketsphinx_continuous -hmm `<your_new_model_folder>` -lm `<your_lm>`
-dict `<your_dict>` -infile test.wav

or with the `-hmm` engine configuration option through the `cmd_ln_init`
function. Alternatively, you can replace the old model files with the new ones.

To use the trained model in Sphinx4, you need to update the model location in
the code.

## Troubleshooting

If the adaptation didn’t improve your results, first test the accuracy and make
sure it’s good.

**I have no idea where to start looking for the problem…**

 1. Test whether the accuracy on the adaptation set improved
 2. Accuracy improved on adaptation set ⇢ check if your adaptation set matches
 		with your test set
 3. Accuracy didn't improve on adaptation set ⇢ you made a mistake during the
	  adaptation

**…or how much improvement I might expect through adaptation**

From few sentences you should get about 10% relative WER improvement.

**I’m lost about…**

…whether it needs more/better training data, whether I'm not doing the
adaptation correctly, whether my language model is the problem here, or whether
there is something intrinsically wrong with my configuration.

Most likely you just ignored some error messages that were printed to you. You
obviosly need to provide more information and give access to your experiment
files in order to get more definite advise.

## What’s next

We hope the adapted model gives you acceptable results. If not, try to improve
your adaptation process by:

 1.  Adding more adaptation data
 2.  Adapting your language mode / using a better language model

 <span class="post-bottom-nav">
 	[Building a language model](/wiki/tutoriallm)
  [Training an acoustic model](/wiki/tutorialam)
 </span>
