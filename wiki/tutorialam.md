---
layout: page
title: Training an acoustic model for CMUSphinx
---

---
* auto-gen TOC:
{:toc}
---

## Introduction

The CMUSphinx project comes with several high-quality acoustic models. There
are US English acoustic models for microphone and broadcast speech as
well as a model for speech over a telephone. You can also use French or
Chinese models trained on a huge amount of acoustic data. Those models
were carefully optimized to achieve the best recognition performance and
work well for almost all applications. We put years of experience into
making them perfect. Most command-and-control applications and even
some large vocabulary applications could just use default models directly.

Besides models, CMUSphinx provides some approaches for adaptation which should
suffice for most cases when more accuracy is required. Adaptation is
known to work well when you are using different recording environments
(close-distance or far microphone or telephone channel), or when a
slightly different accent (UK English or Indian English) or even another
language is present. Adaptation, for example, works well if you
need to quickly add support for some new language just by mapping
a phoneset of an acoustic model to a target phoneset with the dictionary.

There are, however, applications where the current models won't work.
Such examples are handwriting recognition or dictation support for another
language. In these cases, you will need to train your own model and this
tutorial demonstrates how to do the training for the CMUSphinx speech
recognition engine. Before starting with the training make sure you are
familiar with the concepts, prepared the language model. Be sure that you
indeed need to train the model and that you have the resources to do that.

### When you need to train

You need to train an acoustic model if:

* You want to create an acoustic model for a new language or dialect
* OR you need a specialized model for a small vocabulary application
* *AND you have plenty of data to train on*:
  * 1 hour of recording for command and control for a single speaker
  * 5 hours of recordings of 200 speakers for command and control for many speakers
  * 10 hours of recordings for single speaker dictation
  * 50 hours of recordings of 200 speakers for many speakers dictation
* AND you have knowledge on the phonetic structure of the language
* AND you have time to train the model and optimize the parameters (~1 month)

### When you don't need to train

You don’t need to train an acoustic model if:

* You need to improve accuracy – do acoustic model adaptation instead
* You do not have enough data – do acoustic model adaptation instead
* You do not have enough time
* You do not have enough experience

Please note that the amounts of data listed here *are required* to train a
model. If you have significantly less data than listed you can not
expect to train a good model. For example, you *cannot* train a model
with 1 minute of speech data.

## Data preparation

The trainer learns the parameters for the models of the sound units using
a set of sample speech signals. This is called a training database. A
selection of already trained databases will also be provided to you.

The database contains information that is required to extract statistics from
the speech in form of the acoustic model.

The trainer needs to be told which sound units you want it to learn the
parameters of, and at least the sequence in which they occur in every
speech signal in your training database. This information is provided
to the trainer through a file called the *transcript file*. It contains
the sequence of words and non-speech sounds in the exact same order as they
occurred in a speech signal, followed by a tag which can be used to
associate this sequence with the corresponding speech signal.

Thus, in addition to the speech signals and transcription file, the
trainer needs to access two dictionaries: one in which legitimate words
in the language are mapped to sequences of sound units (or sub-word units),
and a another one in which non-speech sounds are mapped to corresponding
non-speech or speech-like sound units. We will refer to the former as
the language *phonetic dictionary*. The latter is called the *filler
dictionary*. The trainer then looks into the dictionaries, to derive the
sequence of sound units that are associated with each signal and transcription.

After training, it's mandatory to run the decoder to check the training
results. The Decoder takes a model, tests part of the database and
reference transcriptions and estimates the quality (WER) of the model.
During the testing stage we use the *language model* with the
description of the possible order of words in the language.

To setup a training, you first need to design a training database or download
an existing one. For example, you can purchase a database from
[LDC](https://www.ldc.upenn.edu). You'll have to convert it into a proper format.

A database should be a good representation of what speech you are going
to recognize. For example if you are going to recognize telephone
speech it is preferred to use telephone recordings. If you want to work with
mobile speech, you should better find mobile recordings.
Speech is significantly different across various recording channels.
Broadcast news is different from a phone call. Speech decoded from mp3 is
significantly different from a microphone recording. However, if you
do not have enough speech recorded in the required conditions you should
definitely use any other speech you have. For example you can use
broadcast recordings. Sometimes it make sense to stream through the
telephone codec to equalize the audio. It's often possible to add noise
to training data too.

The database should have recording of enough speakers, a variety of recording
conditions, enough acoustic variations and all possible linguistic sentences.
As mentioned before, The size of the database depends on the complexity of the
task you want to handle.

A database should have the two parts mentioned above: a training part and a
test part. Usually the test part is about 1/10th of the total data size, but
we don't recommend you to have more than 4 hours of recordings as test data.

Good approaches to obtain a database for a new language are:

* Manually segmenting audio recordings with existing transcription (podcasts, news, etc.)
* Recording your friends and family and colleagues
* Setting up an automated collection on [Voxforge](http://voxforge.org)

You have to design database prompts and post-process the results to
ensure that the audio actually corresponds to the prompts. The file structure
for the database is the following:

    ├─ etc
    │  ├─ your_db.dic                 (Phonetic dictionary)
    │  ├─ your_db.phone               (Phoneset file)
    │  ├─ your_db.lm.DMP              (Language model)
    │  ├─ your_db.filler              (List of fillers)
    │  ├─ your_db_train.fileids       (List of files for training)
    │  ├─ your_db_train.transcription (Transcription for training)
    │  ├─ your_db_test.fileids        (List of files for testing)
    │  └─ your_db_test.transcription  (Transcription for testing)
    └─ wav
       ├─ speaker_1
       │   └─ file_1.wav              (Recording of speech utterance)
       └─ speaker_2
          └─ file_2.wav

Let's go through the files and describe their format and the way to prepare
them:

**\*.fileids:** The `your_db_train.fileids` and `your_db_test.fileids` files
are text files which list the names of the recordings (utterance ids) one by
one, for example:

```
speaker_1/file_1
speaker_2/file_2
```

A *\*.fileids* file contains the path in a file-system relative to the *wav*
directory. Note that a *\*.fileids* file should not include audio file
extensions in its content, but rather just the names.

**\*.transcription:** The `your_db_train.transcription` and
`your_db_test.transcription` files are text files listing the transcription
for each audio file:

```
<s> hello world </s> (file_1)
<s> foo bar </s> (file_2)
```

It's important that each line starts with `<s>` and ends with `</s>`
followed by an id in parentheses. Also note that the parentheses contains only
the file,  without the *speaker_n* directory. It's critical to have an exact
match between the *\*.fileids* file and the *\*.transcription* file.
The number of lines in both should be identical. The last part of the file id
`(speaker1/file_1)` and the utterance id `file_1` must be the same on each line.

Below is an example of an *incorrect* *\*.fileids* file for the above transcription
file. If you follow it, you will get an error as discussed
[here](https://sourceforge.net/p/cmusphinx/discussion/help/thread/278ee211/):

```
speaker_2/file_2
speaker_1/file_1
// Bad! Do not create *.fileids files like this!
```

**Speech recordings (*.wav files):** Your audio recordings should contain
training audio which should match the audio you want to recognize in the end.
In case of a mismatch you could experience a sometimes even significant drop
of the accuracy. This means if you want to recognize continuous speech, your
training database should record continuous speech.
For continuous speech the optimal length for audio recordings is between 5
seconds and 30 seconds. Very long recordings make training much harder.
If you are going to recognize short isolated commands, your training database
should also contain files with short isolated commands. It is better to design
the database to recognize continuous speech from the beginning though and not
spend your time on commands. In the end you speak continuously anyway.
The Amount of silence in the beginning and in the end of the utterance should
not exceed 200 ms.

Recording files must be in *MS WAV* format with a specific sample rate – 16
kHz, 16 bit, mono for desktop application, 8kHz, 16bit, mono for telephone
applications. It’s *critical* that the audio files have a specific format.
Sphinxtrain does support some variety of sample rates but by default it is
configured to train from *16khz 16bit mono* files in MS WAV format.

**So, please make sure that your recordings have a *samplig rate of 16 kHz* (or 8 kHz if you train a telephone model) in *mono* with a *single channel*!**

If you train from an 8 kHz model you need to make sure you configured the
feature extraction properly. Please note that you *cannot upsample* your audio,
that means you can not train 16 kHz model with 8 kHz data.

A mismatch of the audio format is the most common training problem – make sure
you eliminated this source of problems.

**Phonetic Dictionary (your_db.dict):** should have one line per word with
the word following the phonetic transcription:

```
HELLO HH AH L OW
WORLD W AO R L D
```

If you need to find a phonetic dictionary, have a look on Wikipedia or read a
book on phonetics. If you are using an existing phonetic dictionary do not use
case-sensitive variants like "e" and "E". Instead, all your phones must
be different even in the case-insensitive variation. Sphinxtrain doesn’t
support some special characters like "\*" or "/" and supports most of
others like "+", "-" or ":". However, to be safe we recommend you to use
alphanumeric-only phone-set.

Replace special characters in the phone-set, like colons, dashes or
tildes, with something alphanumeric. For example, replace "a~" with
"aa" to make it alphanumeric only. Nowadays, even cell phones have
gigabytes of memory on board. There is no sense in trying to save space
with cryptic special characters.

There is one very important thing here. For a large vocabulary database,
the phonetic representation is more or less known; it’s simple phones
described in any book. If you don’t have a phonetic book, you can just
use the word’s spelling and it will also give you very good results:

```
ONE O N E
TWO T W O
```

For a small vocabulary CMUSphinx is different from other toolkits. It’s
often recommended to train word-based models for a small vocabulary
databases like digits. Yet, this only makes sense if your HMMs could have
variable length.

*CMUSphinx does not support word models.* Instead, you need to use a
word-dependent phone dictionary:

```
ONE W_ONE AH_ONE N_ONE
TWO T_TWO UH_TWO
NINE N_NINE AY_NINE N_END_NINE
```

This is actually *equivalent* to word-based models and some times even
gives better accuracy. *Do not use word-based models with CMUSphinx!*

**Phoneset file (your_db.phone):** should have one phone per line. The
number of  phones should match the phones used in the dictionary plus
the special SIL phone for silence:

```
AH
AX
DH
IX
```

**Language model file (your_db.lm.DMP):** should be in ARPA format or in
DMP format. Find our more about language models in the
[Building a language model](/wiki/tutoriallm) chapter.

**Filler dictionary (your_db.filler):** contains filler phones (not-covered by
language model non-linguistic sounds like breath, "hmm" or laugh).
It can contain just silences:

```
<s> SIL
</s> SIL
<sil> SIL
```

It can also contain filler phones if they are present in the database
transcriptions:

```
+um+ ++um++
+noise+ ++noise++
```

The sample database for training is available on GitHub at
[https://github.com/cmusphinx/an4](https://github.com/cmusphinx/an4).
You can use this database in the following sections. If you want to
play with a large example, download the
[TED-LIUM](http://www-lium.univ-lemans.fr/en/content/ted-lium-corpus)
English acoustic database. It contains about 200 hours of audio
recordings at present.

## Compilation of the required packages

The following packages are required for training:

* sphinxbase
* pocketsphinx

The following external packages are also required:

*  perl, for example ActivePerl on Windows
*  python, for example ActivePython on Windows

In addition, if you download the packages with a `.gz` suffix, you will need
`gunzip` or an equivalent tool to unpack them.

Install the perl and python packages somewhere in your executable path, if they
are not already there.

We recommend that you train on Linux: this way you’ll be able to use all the
features of sphinxtrain. You can also use a Windows system for training,
in that case we recommend to use ActivePerl.

For further download instructions, see
the [download page](http://cmusphinx.github.io/wiki/download/).

Basically you need to put everything into a single root folder, unzip and
untar them, and run `configure` and `make` and `make install` in each
package folder. Put the database folder into this root folder as well.
By the time you finish this, you will have a tutorial directory with
the following contents:

```
└─ tutorial
   ├─ an4
   ├─ an4_sphere.tar.gz
   ├─ sphinxtrain
   ├─ sphinxtrain-5prealpha.tar.gz
   ├─ pocketsphinx
   ├─ pocketsphinx-5prealpha.tar.gz
   ├─ sphinxbase
   └─ sphinxbase-5prealpha.tar.gz
```

You will need to install the software as an administrator `root`. After you
installed the software you may need to update the system configuration
so that the system will be able to find the dynamic libraries, e.g.:

```
export PATH=/usr/local/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/lib
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
```

If you don’t want to install into your system path, you may install the packages
in your home folder. In that case you can append the following option to the
`autogen.sh` script or to the `configure` script:

```
--prefix=/home/user/local
```

Obviously, the folder can be an arbitrary folder, just remember to update
the environment configuration after modifying its name. If your binaries fail
to load dynamic libraries with an error message like `failed to open
libsphinx.so.0 no such file or directory`, it means that you didn't
configure the environment properly.

## Setting up the training scripts

To start the training, change to the database folder and run the following
commands:

*On Linux:*

```
sphinxtrain -t an4 setup
```

*On Windows:*

```
python ../sphinxtrain/scripts/sphinxtrain -t an4 setup
```

Do not forget to replace *an4* with your task name.

This will copy all the required configuration files into the *etc/* subfolder of your
database folder and will prepare the database for training.
The directory structure after the setup will look like this:

```
├─ etc
└─ wav
```

In the process of the training other data folders will be created, so that your
database directory should look like this:

```
├─ etc
├─ feat
├─ logdir
├─ model_parameters
├─ model_architecture
├─ result
└─ wav
```

After this basic setup, we need to edit the configuration files in the *etc/*
folder. There are many variables but to get started we need to change only a
few. First of all, find the file `etc/sphinx_train.cfg`.

### Setting up the format of database audio

In `etc/sphinx_train.cfg` you should see the following configurations:

```
$CFG_WAVFILES_DIR = "$CFG_BASE_DIR/wav";
$CFG_WAVFILE_EXTENSION = 'wav';
$CFG_WAVFILE_TYPE = 'mswav'; # one of nist, mswav, raw
```

### Configuring file paths

Search for the following lines in your `etc/sphinx_train.cfg` file:

```
# Variables used in main training of models
$CFG_DICTIONARY     = "$CFG_LIST_DIR/$CFG_DB_NAME.dic";
$CFG_RAWPHONEFILE   = "$CFG_LIST_DIR/$CFG_DB_NAME.phone";
$CFG_FILLERDICT     = "$CFG_LIST_DIR/$CFG_DB_NAME.filler";
$CFG_LISTOFFILES    = "$CFG_LIST_DIR/${CFG_DB_NAME}_train.fileids";
$CFG_TRANSCRIPTFILE = "$CFG_LIST_DIR/${CFG_DB_NAME}_train.transcription"
```

These values would be already set if you set up the file structure like
described earlier, but make sure that your files are really named this
way.

The `$CFG_LIST_DIR` variable is the `/etc` directory in your project.
The `$CFG_DB_NAME` variable is the name of your project itself.

### Configuring model type and model parameters

To select the acoustic model type see the
[Acoustic Model Types](/wiki/acousticmodeltypes) article.

```
$CFG_HMM_TYPE = '.cont.'; # Sphinx4, Pocketsphinx
#$CFG_HMM_TYPE  = '.semi.'; # PocketSphinx only
#$CFG_HMM_TYPE  = '.ptm.'; # Sphinx4, Pocketsphinx, faster model
```

Just uncomment what you need. For resource-efficient applications use
semi-continuous models, for best accuracy use continuous models. By
default we use PTM models which provide a nice balance between accuracy
and speed.

```
$CFG_FINAL_NUM_DENSITIES = 8;
```

If you are training continuous models for a large vocabulary and have more
than 100 hours of data, put 32 here. It can be any power of 2: 4, 8,
16, 32, 64.

If you are training semi-continuous or PTM model, use 256 gaussians.

```
# Number of tied states (senones) to create in decision-tree clustering
$CFG_N_TIED_STATES = 1000;
```

This value is the number of senones to train in a model. The more
senones a model has, the more precisely it discriminates the sounds.
On the other hand, if you have too many senones, the model will not be
generic enough to recognize yet unseen speech. That means that the WER will
be higher on unseen data. That’s why it is important to *not overtrain
the models*. In case there are too many unseen senones, the warnings
will be generated in the norm log on stage 50 below:

```
ERROR: "gauden.c", line 1700: Variance (mgau= 948, feat= 0, density=3,
component=38) is less then 0. Most probably the number of senones is too
high for such a small training database. Use smaller $CFG_N_TIED_STATES.
```

The approximate number of senones and the number of densities for a continuous
model is provided in the table below:

| Vocabulary |  Audio in database / hours | Senones | Densities | Example |
| -----------|-------------|---------|-----------|---------|
| 20         | 5           | 200     | 8         | Tidigits Digits Recognition |
| 100        | 20          | 2000    | 8         | RM1 Command and Control |
| 5000       | 30          | 4000    | 16        | WSJ1 5k Small Dictation |  
| 20000      | 80          | 4000    | 32        | WSJ1 20k Big Dictation |  
| 60000      | 200         | 6000    | 16        | HUB4 Broadcast News  |      
| 60000      | 2000        | 12000   | 64        | Fisher Rich Telephone Transcription |

For semi-continuous and PTM models use a fixed number of 256 densities.

Of course you also need to understand that only senones that are present in
transcription can be trained. It means that if your transcription
isn’t  generic enough, e.g. if it’s the same single word spoken by
10.000 speakers 10.000 times you still have just a few senones no matter
how many hours of speech you recorded. In that case you just need a
few senones in the model, not thousands of them.

Though it might seem that diversity could improve the model that’s not the
case. Diverse speech requires some artificial speech prompts and that
decreases the speech naturalness. Artificial models don’t help in
real life decoding. In order to build the best database you need to try
to reproduce the real environment as much as possible. It’s even better to
collect more speech to try to optimize the database size.

It’s important to remember, that *optimal numbers depend on your
database*. To train a model properly, you need to experiment with
different values and try to select the ones which result in the best WER for a
development set. You can experiment with the number of senones and the number of
Gaussian mixtures at least. Sometimes it’s also worth to experiment with
the phoneset or the number of estimation iterations.

### Configuring sound feature parameters

The default for sound files used in Sphinx is a rate of 16 thousand
samples per second (16 KHz). If this is the case, the *etc/feat.params*
file will be automatically generated with the recommended values.

If you are using sound files with a sampling rate of 8 kHz (telephone
audio), you need to change some values in *etc/sphinx_train.cfg*. The
lower sampling rate also means a change in the sound frequency ranges
and the number of filters that are used to recognize speech. Recommended
values are:

```
# Feature extraction parameters
$CFG_WAVFILE_SRATE = 8000.0;
$CFG_NUM_FILT = 31; # For wideband speech it's 40, for telephone 8khz reasonable value is 31
$CFG_LO_FILT = 200; # For telephone 8kHz speech value is 200
$CFG_HI_FILT = 3500; # For telephone 8kHz speech value is 3500
```

### Configuring parallel jobs to speedup the training

If you are on a multicore machine or in a PBS cluster you can run the training
in parallel. The following options should do the trick:

```
# Queue::POSIX for multiple CPUs on a local machine
# Queue::PBS to use a PBS/TORQUE queue
$CFG_QUEUE_TYPE = "Queue";
```

Change the type to "Queue::POSIX" to run on multicore. Then change the number of
parallel processes to run:

```
# How many parts to run Forward-Backward estimation in
$CFG_NPART = 1;
$DEC_CFG_NPART = 1; #  Define how many pieces to split decode in
```

If you are running on an 8-core machine start around 10 parts to fully load the
CPU during training.

### Configuring decoding parameters

Open `etc/sphinx_train.cfg` and make sure the following configurations are set:

```
$DEC_CFG_DICTIONARY     = "$CFG_BASE_DIR/etc/$CFG_DB_NAME.dic";
$DEC_CFG_FILLERDICT     = "$CFG_BASE_DIR/etc/$CFG_DB_NAME.filler";
$DEC_CFG_LISTOFFILES    = "$CFG_BASE_DIR/etc/${CFG_DB_NAME}_test.fileids";
$DEC_CFG_TRANSCRIPTFILE = "$CFG_BASE_DIR/etc/${CFG_DB_NAME}_test.transcription";
$DEC_CFG_RESULT_DIR     = "$CFG_BASE_DIR/result";

# These variables are used by the decoder and have to be defined by the user.
# They may affect the decoder output.

$DEC_CFG_LANGUAGEMODEL  = "$CFG_BASE_DIR/etc/${CFG_DB_NAME}.lm.DMP";
```

If you are training with *an4* please make sure that you changed
*${CFG_DB_NAME}.lm.DMP* to *an4.ug.lm.DMP* since the name of the language
model is different in an4 database:

```
$DEC_CFG_LANGUAGEMODEL  = "$CFG_BASE_DIR/etc/an4.ug.lm.DMP";
```

If everything is OK, you can proceed to training.

## Training

First of all, go to the database directory:

```
cd an4
```

To train, just run the following commands:

*On Linux:*

```
sphinxtrain run
```

*On Windows:*

```
python ../sphinxtrain/scripts/sphinxtrain run
```

and it will go through all the required stages. It will take a few
minutes to train. On large databases, training could take up to a month.

The most important stage is the first one which checks that everything is
configured correctly and your input data is consistent.

*Do not ignore the errors reported on the first 00.verify_all step!*

The typical output during decoding will look like:

```
Baum Welch starting for 2 Gaussian(s), iteration: 3 (1 of 1)
0% 10% 20% 30% 40% 50% 60% 70% 80% 90% 100%
Normalization for iteration: 3
Current Overall Likelihood Per Frame = 30.6558644286942
Convergence Ratio = 0.633864444461992
Baum Welch starting for 2 Gaussian(s), iteration: 4 (1 of 1)
0% 10% 20% 30% 40% 50% 60% 70% 80% 90% 100%
Normalization for iteration: 4
```

These scripts process all required steps to train the model. After they
finished, the training is complete.

## Training internals

This section describes in detail what happens during the training.

In the scripts directory (`./scripts_pl`), there are several directories
numbered sequentially from *00* through *99*. Each directory either has a
directory named `slave*.pl` or it has a single file with the extension `.pl`.
The script sequentially goes through the directories and executes either the
`slave*.pl` or the single `.pl` file, as below.

```
perl scripts_pl/000.comp_feat/slave_feat.pl
perl scripts_pl/00.verify/verify_all.pl
perl scripts_pl/10.vector_quantize/slave.VQ.pl
perl scripts_pl/20.ci_hmm/slave_convg.pl
perl scripts_pl/30.cd_hmm_untied/slave_convg.pl
perl scripts_pl/40.buildtrees/slave.treebuilder.pl
perl scripts_pl/45.prunetree/slave-state-tying.pl
perl scripts_pl/50.cd_hmm_tied/slave_convg.pl
perl scripts_pl/90.deleted_interpolation/deleted_interpolation.pl
```

These scripts launch jobs on your machine, and the jobs will take a few
minutes each to run through.

Before you run any script, note the directory contents of your current
directory. After you run each `slave.pl` look at the the contents again.
Several new directories  will have been created. These directories contain
files which are being generated in the course of your training.
At this point you don’t need to know about the contents of these directories,
though some of the directory names may be self-explanatory and you may
explore them if you are curious.

One of the files that appears in your current directory is an *.html*
file, named *an4.html*, depending  on which database you are using. This
file will contain a status report of the already executed jobs. Verify that
the job you launched completed successfully. Only then launch the next
*slave.pl* in the specified order. Repeat this process until you have
run the *slave.pl* in all directories.

Note that in the process of going through the scripts from 00 to 90,
you will have generated  several sets of acoustic models, each of which
could be used for recognition. Notice also that some of the steps are
required only for the creation of semi-continuous models. If you execute
these steps while creating continuous models, the scripts will benignly
do nothing.

In the stage `000.comp_feat` the feature files are extracted. The system
does not directly work with acoustic signals. The signals are first
transformed into a sequence of feature vectors, which are used in place
of the actual acoustic signals.

This script `slave_feat.pl` will compute a sequence of 13-dimensional vectors
(feature vectors) for each training utterance consisting of the
Mel Frequency Cepstral Coefficients
([MFCCs](https://de.wikipedia.org/wiki/Mel_Frequency_Cepstral_Coefficients)).
Note that the list of wave files contains a list with the full paths to the
audio files. Since the data is all located in the same directory as your working
directory, the paths are relative, not absolute. If the location of data is
different, you may have to change this, as well as the *an4_test.fileids* file.
The MFCCs will be placed automatically in a directory called `feat`.
Note that the type of features vectors you compute from the speech signals
for training and recognition, outside of this tutorial, is not
restricted to MFCCs. You could use any reasonable parameterization
technique instead, and compute features other than MFCCs. CMUSphinx can
use features of any type or dimensionality. The format of the features
is described on the [MFC Format](/wiki/mfcformat) page.

Once the jobs launched from `20.ci_hmm` have run to completion, you will
have trained the *Context-Independent* (CI) models for the sub-word units
in your dictionary.

When the jobs launched from the `30.cd_hmm_untied` directory run to
completion, you will have trained the models for *Context-Dependent*
sub-word units (triphones) with untied states. These are called
*CD-untied models* and are necessary for building decision trees in
order to tie states.

The jobs in `40.buildtrees` will build decision trees for each state of
each sub-word unit.

The jobs in `45.prunetree` will prune the decision trees and tie the states.

Following this, the jobs in `50.cd-hmm_tied` will train the final models
for the triphones in your training corpus. These are called *CD-tied models*.
The CD-tied models are trained in many stages. We begin with 1 Gaussian per
state HMMs, followed by training 2 Gaussian per state HMMs and so on till the
desired number of Gaussians per State have been trained.
The jobs in `50.cd-hmm_tied` will automatically train all these intermediate
CD-tied models.

At the end of any stage you may use the models for recognition. Remember
that you may decode even while the training is in progress, provided you are
certain that you have crossed the stage which generates the models you want to
decode  with.

### Transformation matrix training (advanced)

Some additional scripts will be launched if you choose to run them.
These additional training steps can be costly in computation, but improve the
recognition rate.

Transform matrices might help the training and recognition process in some
circumstances.

The following steps will run if you specify

```
$CFG_LDA_MLLT = 'yes';
```

in the  file `sphinx_train.cfg`. If you specify `'no'` (which is the default),
the steps will do nothing. The Step `01.lda_train` will estimate a LDA matrix
and the step `02.mllt_train` will estimate a MLLT matrix.

The Perl scripts, in turn, set up and run Python modules. The end product for
these steps is a `feature_transform` file,  in your `model_parameters` directory.
For details see the page for [Training with LDA and MLLT](/wiki/ldamllt).

### MMIE training (advanced)

Finally, one more step will run if you specify MMIE training by setting:

```
$CFG_MMIE =  "yes";
```
The default value is `"no"`. This will run steps `60.lattice_generation`, `61.lattice_pruning`, `62.lattice_conversion` and `65.mmie_train`.
For details see the page about [MMIE Training in SphinxTrain](/wiki/mmie_train).

## Testing

It's *critical* to test the quality of the trained database in order to
select the best parameters, understand how your application performs and
optimize the performance. To do that, a test decoding step is needed. The decoding
is now a last stage of the training process.

You can restart decoding with the following command:

```
sphinxtrain -s decode run
```

This command will start a decoding process using the acoustic model you
trained and the language model you configured in the `etc/sphinx_train.cfg` file.

```
MODULE: DECODE Decoding using models previously trained
Decoding 130 segments starting at 0 (part 1 of 1)
0%
```

When the recognition job is complete, the script computes the
recognition Word Error Rate (WER) and the  Sentence Error Rate (SER).
The lower those rates the better is your recognition. For a typical 10-hours
task the WER should be around 10%. For a large task, it could be like 30%.

On an4 data you should get something like:

```	 
SENTENCE ERROR: 70.8% (92/130)   WORD ERROR RATE: 30.3% (233/773)
```

You can find exact details of the decoding, like the alignment with
a reference transcription, speed and the result for each file, in the `result`
folder which will be created after decoding.
Let’s have a look into the file `an4.align`:

```
p   I   T      t   s   b   u   r   g   H      (MMXG-CEN5-MMXG-B)
p   R   EIGHTY t   s   b   u   r   g   EIGHT  (MMXG-CEN5-MMXG-B)
Words: 10 Correct: 7 Errors: 3 Percent correct = 70.00% Error = 30.00% Accuracy = 70.00%
Insertions: 0 Deletions: 0 Substitutions: 3
october twenty four nineteen seventy  (MMXG-CEN8-MMXG-B)
october twenty four nineteen seventy  (MMXG-CEN8-MMXG-B)
Words: 5 Correct: 5 Errors: 0 Percent correct = 100.00% Error = 0.00% Accuracy = 100.00%
Insertions: 0 Deletions: 0 Substitutions: 0
TOTAL Words: 773 Correct: 587 Errors: 234
TOTAL Percent correct = 75.94% Error = 30.27% Accuracy = 69.73%
TOTAL Insertions: 48 Deletions: 15 Substitutions: 171
```

For a description of the WER see our
[Basic concepts of speech](wiki/tutorialconcepts/#what-is-optimized) chapter.

## Using the model

After training, the acoustic model is located in

```
model_parameters/`<your_db_name>`.cd_cont_`<number_of senones>`
```

or in

```
model_parameters/`<your_db_name>`.cd_semi_`<number_of senones>`
```

You need only that folder. The model should have the following files:

```
mdef
feat.params
mixture_weights
means
noisedict
transition_matrices
variances
```

depending on the type of the model you trained. To use the model in PocketSphinx,
simply point to it with the `-hmm` option:

```
pocketsphinx_continuous -hmm `<your_new_model_folder>` -lm `<your_lm>` -dict `<your_dict>`.
```

To use the trained model in Sphinx4, you need to specify the path in the
`Configuration` object:

```java
configuration.setAcousticModelPath("file:model_parameters/db.cd_cont_200");
```

If the model is in the resources you can reference it with `resource:URL`:

```java
configuration.setAcousticModelPath("resource:/com/example/db.cd_cont_200");
```

See the [Sphinx4 tutorial](/wiki/tutorialsphinx4) for details.

## Troubleshooting

Troubleshooting is not rocket science. For all issues you may blame
yourself. You are most likely the reason of failure. *Carefully* read
the messages in the `logdir` folder that contains a detailed log for each
performed action. In addition, messages are copied to the
`your_project_name.html` file, which you can open and read in a browser.

There are many well-working, proven methods to solve issues. For example,
try to reduce the training set to see in which half the problem appears.

Here are some common problems:

* **WARNING: this phone (something) appears in the dictionary (dictionary file
  name), but not in the phone list (phone file name).**

  Your dictionary either contains a mistake, or you have left out a phone symbol
  in the phone file. You may have to delete any comment lines from your
  dictionary file.

* **WARNING: This word (word) has duplicate entries in (dictionary file name).
  Check for duplicates.**

  You may have to sort your dictionary file lines to find them. Perhaps a word
  is defined in both upper and lower case forms.

* **WARNING: This word: word was in the transcript file, but is not in the
  dictionary (transcript line) Do cases match?**

  Make sure that all the words in the transcript are in the dictionary, and that
  they have matching cases when they appear. Also, words in the transcript may
  be misspelled, run together or be a number or symbol that is not in the
  dictionary. If the dictionary file is not perfectly sorted, some entries might
  be skipped while looking for words. If you hand-edited the dictionary file, be
  sure that each entry is in the proper format.

  You may have specified phones in the phone list that are not represented in
  the words in the transcript. The trainer expects to find examples of each
  phone at least once.

* **WARNING: CTL file, audio file name.mfc, does not exist, or is empty.**

  The *.mfc* files are the feature files converted from the input audio files in
  stage `000.comp_feats`. Did you skip this step? Did you add new audio files
  without converting them? The training process expects a feature file to be
  there, but it isn’t.

* **Very low recognition accuracy**

  This might happen if there is a mismatch in the audio files and the parameters
  of training, or between the training and the testing.

* **ERROR: "backward.c", line 430: Failed to align audio to transcript: final
  state of the search is not reached.**

  Sometimes audio in your database doesn’t match the transcription properly. For
  example the transcription file has the line "Hello world" but in audio
  actually "Hello hello world" is pronounced. The training process usually
  detects that and emits this message in the logs. If there are too many of such
  errors it most likely means you misconfigured something, e.g. you had a
  mismatch between audio and the text caused by transcription reordering.
  If there are few errors, you can ignore them. You might want to edit the
  transcription file to put the exact word which was pronounced. In the case
  above you need to edit the transcription file and put "Hello hello world" on
  the corresponding line. You might want to filter such prompts because they
  affect the quality of the acoustic model. In that case you need to enable
  the forced alignment stage during training. To do that edit the following line in `sphinx_train.cfg`:
  ```
  $CFG_FORCEDALIGN = 'yes';
  ```
  and run the training again. It will execute stages 10 and 11 and will filter
  your database.

* **Can't open \*/\*-1-1.match word_align.pl failed with error code 65280**

  This error occurs because the decoder did not run properly after the training.
  First check if the correct executable is present in your `PATH`.
  The executable shouldbe `pocketsphinx_batch` if the decoding script being used
  is `psdecode.pl` as set by the `$DEC_CFG_SCRIPT` variable in
  `sphinx_train.cfg`. On Linux run:

  ```
  which pocketsphinx_batch
  ```

  and see if it is located as expected. If it is not, you need to set the `PATH`
  variable properly. Similarly on Windows, run:

  ```
  where pocketsphinx_batch
  ```

  If the path to the decoding executable is set properly, read the log files in
  `logdir/decode/` to find out other reasons for the error.

* **To ask for help**

  If you want to ask for help about training, try to provide the training
  folder or at least the logdir. Pack the files into an archive and upload it
  to a public file sharing resource. Then post the link to the resource.
  Remember: the more information you provide the faster you will solve the
  problem.

<span class="post-bottom-nav">
  [Adapting an existing acoustic model](/wiki/tutorialadapt)
  [Tuning the performance](/wiki/tutorialtuning)
</span>
